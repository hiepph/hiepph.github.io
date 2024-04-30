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

Site is automatically deployed with [GitHub Actions][0] and [actions-hugo][3] module.

Generated site and assets are stored in `gh-pages` branch.

1. Go to "Settings > Pages". In "Build and deployment", choose "Deploy from a branch" source.
2. Choose `gh-pages` branch, `/ (root)` folder.
3. Leave custom domain empty, as it will be populated with [peaceiris/actions-gh-pages][6] module.

## Configurations

+ `config.toml`: general Hugo configurations. Values here will overwrite theme's `config.toml`.
+ `data/menu.toml`: information displayed in the home page.

## Custom domains

My blog is reachable via [hiepph.xyz][4], provided that this repository is **public**.

The configuration for the GitHub Pages with custom domain can be referred [here][2].

It's important to update these values when having a new domain:

+ `baseURL` in `config.toml`
+ `cname` in `.github/workflows/gh-pages.yml`


### How it works

I registered my domain `hiepph.xyz` on [AWS Route53][1]. [DNSSEC][7] is also available.

All traffic to `hiepph.xyz` first reaches GitHub Pages' IPs, via `A` records. These records are configured with AWS Route53.

``` sh
nslookup hiepph.xyz
```

```
Non-authoritative answer:
Name:	hiepph.xyz
Address: 185.199.108.153
Name:	hiepph.xyz
Address: 185.199.109.153
Name:	hiepph.xyz
Address: 185.199.110.153
Name:	hiepph.xyz
Address: 185.199.111.153
```

SSL/TLS certificate is automatically generated (from Let's Encrypt) and managed by GitHub Pages ([reference][8]).

Traffic is then redirected to `www.hiepph.xyz`. The redirect is handled automatically with `www` domain and on the server-side of GitHub Pages—you can only see `200` response. 

``` sh
curl --head https://hiepph.xyz
```

```
HTTP/2 200
server: GitHub.com
content-type: text/html; charset=utf-8
```

`www.hiepph.xyz` is a `CNAME` record, pointing to `hiepph.github.io`.

``` sh
nslookup www.hiepph.xyz

```

```
Non-authoritative answer:
www.hiepph.xyz	canonical name = hiepph.github.io.
Name:	hiepph.github.io
Address: 185.199.110.153
Name:	hiepph.github.io
Address: 185.199.111.153
Name:	hiepph.github.io
Address: 185.199.108.153
Name:	hiepph.github.io
Address: 185.199.109.153
```

That's why you can access my blog via [hiepph.github.io][5] as well.



[0]: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
[1]: https://aws.amazon.com/route53/
[2]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
[3]: https://github.com/peaceiris/actions-hugo
[4]: https://hiepph.xyz
[5]: https://hiepph.github.io
[6]: github.com/peaceiris/actions-gh-pages
[7]: https://www.cloudflare.com/en-gb/dns/dnssec/how-dnssec-works/
[8]:  https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https#troubleshooting-certificate-provisioning-certificate-not-yet-created-error
