# Introduction to Kubernetes (through hands-on)

This repository aims to quickly learn the basics of [Kubernetes](https://kubernetes.io/).

:warning: None of the examples given here are made for production.

## Requirements

* docker
* [k3d](https://k3d.io/) >4.x.x
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)

## Agenda

* **[Prepare your local Kubernetes environment](docs/01_local_kubernetes.md)**
* **[Run an application on Kubernetes](docs/02_run_an_application.md)**
* **[Deploy a Wordpress](docs/03_wordpress.md)**
* **[Resources and autoscaling](docs/04_resources_autoscale.md)**
* **[Troubleshooting](docs/05_troubleshooting.md)**
* **[RBAC](docs/06_rbac.md)**

## Cleanup

Pretty simple we’ll drop the whole k3d cluster

```console
$ k3d cluster delete workshop
INFO[0000] Deleting cluster 'workshop'
...
INFO[0008] Successfully deleted cluster workshop!
```

:arrow_right: You may want to continue with the [Helm workshop](https://github.com/Smana/workshop_helm_introduction_2021)
