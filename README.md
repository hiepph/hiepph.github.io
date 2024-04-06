# Personal blog


## Writing locally

+ Pull the repo and theme:

```
git clone --recurse-submodules git@github.com:hiepph/hiepph.github.io.git
```

+ Write new posts in `content/posts` and store static images in `static`. Put `toc: true` to render a Table of Contents.

+ Run the local server:

```
hugo server
```


## Deployment

Site is automatically deployed with [Github Actions][0] and [actions-hugo][3] module.

Generated site and assets are stored in `gh-pages` branch.

1. Go to "Settings > Pages". In "Build and deployment", choose "Deploy from a branch" source.
2. Choose `gh-pages` branch, `/ (root)` folder.
3. Leave custom domain empty, as it will be populated with [peaceiris/actions-gh-pages][6] module.

## Configurations

+ `config.toml`: general Hugo configurations. Values here will overwrite theme's `config.toml`.
+ `data/menu.toml`: information displayed in the home page.

## Domains

My blog is reachable via [hiepph.github.io][5] or [hiepph.xyz][4], provided that this repository is **public**.

I registered my domain `hiepph.xyz` on [AWS Route53][1]. [DNSSEC][7] is also available.

All traffic to `hiepph.xyz` first reaches Github Pages' IP (via `A`/`AAAA` records). They are then redirected to `www.hiepph.xyz`, and finally routed to `hiepph.github.io` (via `CNAME` record).

The configuration for the Github Pages with custom domain can be referred [here][2].

It's important to update these values when having a new domain:

+ `baseURL` in `config.toml`
+ `cname` in `.github/workflows/gh-pages.yml`


[0]: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
[1]: https://aws.amazon.com/route53/
[2]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
[3]: https://github.com/peaceiris/actions-hugo
[4]: https://hiepph.xyz
[5]: https://hiepph.github.io
[6]: github.com/peaceiris/actions-gh-pages
[7]: https://www.cloudflare.com/en-gb/dns/dnssec/how-dnssec-works/
