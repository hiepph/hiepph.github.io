# Personal blog


## Local Writing

+ Pull the repo and theme:

```
git clone --recurse-submodules git@github.com:hiepph/hiepph.github.io.git
```

+ Push new post in `content/post` and static images in `static`

+ Run the local server:

```
hugo server
```


## Deployment

Site is automatically deployed with Github Actions and the supported
[module](https://github.com/peaceiris/actions-hugo).


## Domains

My blog is reachable via https://hiepph.github.io or https://hiepph.xyz.

I registered my domain `hiepph.xyz` on [AWS Route53][1]. All traffic to
`hiepph.xyz` will first reach Github Pages' IP (via `A`/`AAAA` records), be
automatically redirected to `www.hiepph.xyz`, and then finally be routed to
`hiepph.github.io` (via `CNAME` record).

The configuration for the Github Pages with custom domain can be referred
[here][2].


---
[1]: https://aws.amazon.com/route53/
[2]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
