---
title: "Automate all the things"
date: 2021-06-11
---

It's been a while since my last post.

So [FVI](/post/2018-11-13-ocr) used to be a single service specific for reading ID card
information. But now, it's growing to a multi-service platform with
tons of other form-constraint documents. The process is still the
same. We have to duplicate it many more times in a much
faster and efficient way. The problem is turning to automation.

My team and I have been pondering many more technologies outside of the machine learning world to tackle the problem.
So I thought, why I don't share some practices I learn along the way.
This post is more related to DevOps and technology choice decision.


# Ansible

The first thing I look into is how we can bootstrap the platform
quickly and efficiently. [Ansible](https://www.ansible.com/) suits our need.

In a nutshell, Ansible is a tool for configuration management.
I bootstrap what packages I need, what process to run to properly set
up a brand new server to support our platform architecture.
You can think of a bash script file, but written in YAML and is run remotely from your local machine through SSH.

For example, I need to set up Docker and Docker-compose and some
necessary directories for the platform on a Ubuntu server.

![ansible](/automation/ansible.png)

The sample code for setting up docker would be (`setup.yaml`):

```yaml
- name: Set up docker
  gather_facts: no
  hosts: all
  tags: docker
  become: yes
  tasks:
    - name: Install aptitude using apt
      apt: name=aptitude state=latest update_cache=yes force_apt_get=yes

    - name: Install required system packages
      apt: name={{ item }} state=present update_cache=yes
      loop: ['apt-transport-https', 'ca-certificates', 'curl', 'software-properties-common', 'python3-pip', 'virtualenv', 'python3-setuptools']

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Update apt and install docker-ce
      apt: update_cache=yes name=docker-ce state=present

    - name: Install Docker Compose
      pip:
        name: docker-compose

    - name: Add user to docker group
      shell: usermod -aG docker {{ ansible_user }}
```

The code is self-explanatory. Ansible urges us to
switch to the mindset of treating our **infrastructure as code**.
Scripts are managed with version control, have documentation and contribution between team members.

I can manage my servers through a hosts file (`/etc/ansible/hosts`):

```txt
[local]
localhost    ansible_connection=local

[prod]
<ip1>    ansible_user=hiepph
<ip2>    ansible_user=hiepph
```

Then I can quickly run the below command to set up `docker`, and
`docker-compose` on both servers that matched the `group` prod.
With the support of [idempotence](https://en.wikipedia.org/wiki/Idempotence),
I can limit the side effects in installing process. No sweat!

```
ansible-playbook setup.yaml -l prod -t docker
```

Ansible provides thousand of plugins, and I use one to manage cron jobs efficiently.
For example, I manage the repeated tasks of backup, clearing temporary data or cache, etc.
A concept using `rclone` to backup hourly is demonstrated below.

```yaml
- name: backup concept using rclone
  cron:
    special_time: "hourly"
    name: "backup the precious ring"
    job: "rclone copy /precious/ backup-cloud:/precious"
```


# CI/CD

![automate](https://raw.githubusercontent.com/RogueDudes/roguedudes.github.io/master/assets/images/automate.jpg)

There are several significant reasons to bring [CI/CD](https://www.wikiwand.com/en/CI/CD) to the table.

We have to do the same thing over and over again:  building the image, deploy the new build to the production, pulling new models, etc. These repetitive tasks suck. They drain your energy from thinking what the script for this task, which documentation and
the process you have to look into before doing anything.
The mantra is **Define then Forget**. You define the automation
process (script) then forget all about it to reserver energy for other tasks.

Human resource is valuable. The brain is for ideas, not for storage. So be
wise where to put energy.

It's normal for human to make mistakes. However, with a predefined
process, the machine is much less likely to make a mistake.
Here you contain the problem into a box and know where to look for
when the bugs happen.

One huge benefit we notice immediately is the velocity of deployment, both in staging and production environment.

![ci-cd](/automation/ci-cd.png)

For example, I define the following deploy process:

1. A member writes a new feature. He first runs the local test, then submits a merge request on the repository. After merging code, he proceeds to tag a version to deploy.
2. When the process is triggered, it first builds a new image with the same tag. Next, the image is deployed in a test environment. The test is then triggered. If any bug happens, the process fails. So the member has to fix it and repeat the process. Otherwise, the build is deployed in the staging/production environment depending on the tag's pattern.

To achieve the CI/CD pipeline, I utilize the [Gitlab Runner](https://docs.gitlab.com/runner/) on our
self-hosted Gitlab.
I'm still in awe of how easy to adopt [CI/CD](https://docs.gitlab.com/ee/ci/) using Gitlab.
Kudos to the team!

To set up CI/CD, you can follow the Gitlab Runner's installation
guide. Or you can use *Ansible*
[role](https://github.com/riemers/ansible-gitlab-runner)
to easily set one up. I prefer the latter since we have many servers to manage.

First the requirement (`requirements.yaml`):

```yaml
roles:
  - riemers.gitlab-runner
```

Then you can use `ansible-galaxy` to install the role:

```
ansible-galaxy install -r requirements.yml
```

A sample playbook (`gitlab.yaml`):

```yaml
- name: Setup Gitlab Runner
  hosts: all
  become: yes
  vars_files:
    - vars/runner.yaml
  roles:
    - { role: riemers.gitlab-runner }
  tasks:
    - name: add gitlab-runner to groups
      shell: usermod -aG {{ item }} gitlab-runner
      with_items:
        - {{ ansible_user }}
        - docker
```

Inside `vars`, I provide some tokens and settings:

```yaml
gitlab_runner_coordinator_url: "<gitlab-domain>"
gitlab_runner_registration_token: "<token>"
gitlab_runner_runners:
  - name: "ocr-prod"
    executor: "shell"
    state: "present"
    tags:
      - prod
    docker_privileged: true
```

Then one command to set up:

```shell
ansible-playbook gitlab.yml -l prod
```

Now my main job is to maintain the `.gitlab-ci.yml` file to define the
pipeline. Below is the concept code:

```yaml
variables:
    IMAGE: <image-name>
    TEST_DIR: <test-directory>
    STAGING_DIR: <staging-directory>
    PROD_DIR: <prod-directory>

stages:
    - build
    - test
    - deploy

build:
    stage: build
    script:
        - docker build . -t $IMAGE:$CI_COMMIT_TAG
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        - docker push $IMAGE:$CI_COMMIT_TAG

test:
    stage: test
    script:
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        - docker pull $IMAGE:$CI_COMMIT_TAG
        - cd $TEST_DIR
        - docker-compose up -d
        - pytest
        - docker-compose down

staging:
    stage: deploy
    script:
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        - docker pull $IMAGE:$CI_COMMIT_TAG
        - cd $STAGING_DIR
        - docker-compose up -d

prod:
    stage: deploy
    script:
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        - docker pull $IMAGE:$CI_COMMIT_TAG
        - cd $PROD_DIR
        - docker-compose up -d
```

The variables with `$CI_` prefix are predefined variables of our
self-hosted Gitlab code base and image registry. There are many more
predefined variables to support you for the automation process.

With the simple act of tagging, we can achieve 5-10 releases a day. That's a massive step since we used to release weekly, and the process required various members to join.

Due to the policy, I can only reveal the concept and some of our code.
However, I hope with the mindset and these tools; you can adapt automation to your team and daily life.
