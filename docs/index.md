# Welcome to my Homelab documentation

This documentation is where I track my infrastructure setup, how-to's, cheatsheet, experiments and automation projects. My lab runs primarily on a Proxmox cluster with TrueNAS SCALE for storage and Kubernetes (k3s) for running services. I use Ansible to automate deployments and manage various operational tasks. Additionally, I follow a GitOps approach for Kubernetes using a [dedicated repository](https://github.com/x-real-ip/gitops) with ArgoCD to declaratively manage my services and configurations.

The goals of this documentation are twofold:

- To serve as a reference for my own environment and workflows.
- To share ideas, configurations, and lessons learned that may help others building or maintaining their own homelab.

Here, you’ll find notes, guides, and code snippets covering virtualization, storage, and automation, all tested and documented from a real-world setup.


## Serve docs locally

```
podman run --rm -it -v ${PWD}:/docs -p 8000:8000 docker.io/squidfunk/mkdocs-material serve -a 0.0.0.0:8000
```