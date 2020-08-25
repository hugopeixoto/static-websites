## Static websites

Self hosted netlify, kind of?

I host a bunch of static websites on my web server. This helps me keep track of
what's there and a way to automate its setup.

`static-websites` works by adding files to `sites/`, each one describing a website. For
example, I have a file `sites/hugopeixoto.net`:

```yaml
domains: hugopeixoto.net
repository: git@github.com:hugopeixoto/hugopeixoto.net.git
directory: build
build: make
```

This defines that `hugopeixoto.net` should be deployed by cloning my github
repo, calling `make`, and uploading `build`.

To deploy **every** website in `sites/` to a single host, run:

```
./deploy root@example.com
```

Where `example.com` is the address of your server. This assumes you can `ssh
root@example.com`.

## Hardcoded assumptions

There are some hardcoded paths in the scripts. I might make them configurable eventually, but for now:

- Websites will be deployed to `/srv/www/<primary-domain>/public`
- Every website will be configured with HTTPS
- Uses `letsencrypt`, and not `certbot` (I need to upgrade my ubuntu soon)
- Uses `/srv/www/default` as the webroot for letsencrypt certificates
- Assumes you have HTTP requests being redirected to HTTPS, except for `/.well-known/acme-challenge/` paths
- Enables HSTS for every website
- Reloads nginx with `service nginx reload`
- Serves files with charset UTF-8

For reference, here's my `/etc/nginx/sites-enabled/default`:

```nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;
  charset UTF-8;

  root /srv/www/default/;

  location / {
    return 301 https://$host$request_uri;
  }

  location /.well-known/acme-challenge/ {
  }
}
```

That redirection is probably not super safe. It feels like I'm opening myself
to be used as a redirection proxy. If you use this, beware.

## Website description

Each file in `sites/` should be a valid yaml file. While the filename doesn't
matter, my convention is to use the primary domain name.

Here's a list of supported keys:

| Key name | Required | Description |
|---|---|---|
| `domains` | yes | A space separated list of domains of this website. The first domain is considered the primary one and will be used in filenames |
| `redirect` | no | Instead of deploying a website, nginx will redirect every request to the URL in this key |
| `repository` | if redirect is absent | A git clonable URL |
| `build` | no | The command to build the website, if needed. Defaults to `make` if a Makefile is available |
| `directory` | no | The directory to upload to the host, after running the build command. Defaults to `build` |
| `extra_config` | no | Extra nginx configuration. Useful to add rewrites, for example |

### Example descriptions

Redirection only website, multiple domains:

~~~~yaml
domains: lifeonmars.io www.lifeonmars.io
redirect: https://lifeonmars.pt$request_uri
~~~~

Website with no build step:

~~~~yaml
domains: archive.makeorbreak.io
repository: git@github.com:makeorbreak-io/web-archive.git
directory: archive
~~~~

Node.js website:

~~~~yaml
domains: vasteroids.lifeonmars.pt
repository: git@github.com:lifeonmarspt/vasteroids.git
directory: public
build: yarn && yarn run webpack
~~~~

Extra nginx configuration:

~~~~yaml
domains: hugopeixoto.net
repository: git@github.com:hugopeixoto/hugopeixoto.net.git
directory: build
extra_config: |

  try_files $uri.html $uri $uri/index.html =404;

  rewrite ^/blog/feed.xml$ $scheme://$host/articles.xml permanent;
  rewrite ^/blog/(.*).html $scheme://$host/articles/$1.html permanent;
~~~~
