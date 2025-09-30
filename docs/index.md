# Welcome to my Homelab documentation

Welcome to my homelab documentation!
This repository is where I keep track of my infrastructure setup, experiments, and automation work. My lab primarily runs on a Proxmox cluster with TrueNAS SCALE providing storage services. Kubernetes (k3s) to run my services. I use Ansible to automate deployments and manage tasks.

The goal of this documentation is twofold:

- To serve as a reference for my own environment and workflows.

- To share ideas, configurations, and lessons learned that may help others building or maintaining their own homelab.

You’ll find notes, guides, and code snippets related to virtualization, storage, automations, all tested and documented from my real-world setup.


## Serve docs locally

```
podman run --rm -it -v ${PWD}:/docs -p 8000:8000 squidfunk/mkdocs-material serve -a 0.0.0.0:8000
```