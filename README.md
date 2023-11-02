# Personal blog


## Writing locally

+ Pull the repo and theme:

```
git clone --recurse-submodules git@github.com:hiepph/hiepph.github.io.git
```

+ Write new posts in `content/posts` and store static images in `static`.

+ Run the local server:

```
hugo server
```


## Deployment

Site is automatically deployed with Github Actions and 
[actions-hugo](https://github.com/peaceiris/actions-hugo) module.

Generated site and assets are stored in `gh-pages` branch.
Go to "Settings > Pages" to see available options. 

## Configurations

+ `config.toml`: general Hugo configurations. Values here will overwrite
  theme's `config.toml`.
+ `data/menu.toml`: information displayed in the home page.

## Domains

My blog is reachable via https://hiepph.github.io or https://hiepph.xyz.

I registered my domain `hiepph.xyz` on [AWS Route53][1]. All traffic to
`hiepph.xyz` will first reach Github Pages' IP (via `A`/`AAAA` records), be
automatically redirected to `www.hiepph.xyz`, and then finally be routed to
`hiepph.github.io` (via `CNAME` record).

The configuration for the Github Pages with custom domain can be referred
[here][2].


[1]: https://aws.amazon.com/route53/
[2]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
