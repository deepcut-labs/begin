# deepcutlabs.com

The Deepcut Labs, LLC website — a static site served by GitHub Pages at
[deepcutlabs.com](https://deepcutlabs.com).

## Layout

```
index.html                        home
about.html                        company details
contact.html                      support, phone, mailing address
products/index.html               product overview
products/agent-configurator.html  Agent Configurator
products/mdview.html              MDview
privacy.html                      privacy policy, both products
404.html                          not found
assets/site.css                   shared stylesheet — every page links it
assets/img/                       product screenshots (max 1400px wide)
downloads/                        notarized direct downloads
```

There is no build step. `.nojekyll` is present, so GitHub Pages serves the files
as they are — the header and footer are hand-copied into each page. All internal
links are root-relative (`/about.html`, `/assets/site.css`) so they resolve the
same from `products/` as from the root.

## Working on it locally

```sh
python3 -m http.server 8000    # then open http://localhost:8000
```

Screenshots go through `sips -Z 1400` before being committed. Anything showing a
real client name, repository path or session transcript does not go in at all.
