# Welcome to my Homelab documentation

## Serve docs locally

    podman run --rm -it -v ${PWD}:/docs -p 8000:8000 squidfunk/mkdocs-material serve -a 0.0.0.0:8000