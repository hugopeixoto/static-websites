## Static websites

Self hosted netlify, kind of?

I host a bunch of static websites on my web server. This helps me keep track of
what's there and a way to automate its setup.

This works by adding files to `sites/`, each one describing a website. For
example, I have a file `sites/hugopeixoto.net`:

```yaml
domains: hugopeixoto.net
repository: git@github.com:hugopeixoto/hugopeixoto.net.git
directory: build
build: make
```

This instructs the scripts to deploy `hugopeixoto.net` by cloning my github
repo, calling `make`, and uploading `build`.

This doesn't do the deploy part yet, and it's hardcoded to diff the generated nginx configuration file against what's currently hosted at `hugopeixoto.net`.
