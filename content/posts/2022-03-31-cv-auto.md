---
title: "CV building automation with Gitlab CI"
date: 2022-03-31
---

I was updating my CV to apply for a job, and my [yak shaving](https://en.wiktionary.org/wiki/yak_shaving) habit led me astray.

I already wrote my CV in LaTeX and uploaded its source to a personal Gitlab repository. So I was thinking let's use [Gitlab CI/CD](https://docs.gitlab.com/ee/ci/) to automate the process of building my CV. There are some main benefits:

1. I can update the LaTeX source everywhere and have Gitlab Runner build my CV without having a LaTeX environment.

2. I want to have a record of my CV releases.

## Strategy

Let's get the idea rolling with a draft diagram:

![diagram](/cv-auto/diagram.jpeg)

1. After the source code is tagged, the pipeline is triggered. Each tag represents a release with format `<date>-<month>-<semantic-version>`. For example, `2022-03-2.2.13` means the CV version 2.2.13 is written in March 2022.

2. Gitlab Runner builds the CV in a LaTeX environment and outputs the PDF version in Gitlab's [artifacts](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html) directory.

3. Reading the `ARTIFACT_URL` environment variable from the build phase, Gitlab Runner creates a [release](https://docs.gitlab.com/ee/user/project/releases/) pointing to my CV's URL.


## Code

Below is my sample `.gitlab-ci.yml` configuration:

```yml
stages:
  - build
  - release

build:
  stage: build
  image: "listx/texlive:2020"
  script:
    - export THIS_MONTH=$(date +%Y_%m)
    - export OUT_CV=Pham_Hoang_Hiep_cv_${THIS_MONTH}_${CI_COMMIT_SHORT_SHA} # NOTE [1]
    - pdflatex -file-line-error --jobname=${OUT_CV} cv/cv.tex
    - echo "ARTIFACT_URL=${CI_PROJECT_URL}/-/jobs/${CI_JOB_ID}/artifacts/file/${OUT_CV}.pdf" >> build.env # NOTE [2]
  artifacts:
    paths:
      - "*.pdf"
    reports:
      dotenv: build.env
  only:
    - tags

release:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  script:
    - echo "Getting artifact from ${ARTIFACT_URL}"
  needs: # NOTE [2]
    - job: build
      artifacts: true
  release:
    name: "$CI_COMMIT_TAG"
    description: "Created using release-cli."
    tag_name: "$CI_COMMIT_TAG"
    assets:
      links:
        - name: "CV"
          url: "${ARTIFACT_URL}" # NOTE [2]
  only:
    - tags
```

Some important notes:

1. Add some salts (e.g. my commit's SHA) to my generated CV in order not to make release URLs conflict. Reference: [here](https://gitlab.com/gitlab-org/release-cli/-/issues/73).

2. Pass the `ARTIFACT_URL` environment to the next stage (release). Otherwise release wouldn't be able to know the created artifact path.

## Result

And here is my result on the *Releases* page.

![release](/cv-auto/release.png)

Do you see the shiny link to my CV in the *Other* section?

That's all folks! Now I should be back to job hunting instead of doing this.
