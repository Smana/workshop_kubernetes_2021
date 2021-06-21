# Kubernetes workshop

:warning: Work in progess

**Todo**
* ServiceAccounts and RBAC (create a serviceaccount for an external application)
* Persistent volumes (deploy a database)
* Deployment, rolling update, crashloopback, logs, autoscaling
* Jobs, cronjobs
* Go client, Python client

## Requirements

* docker
* [k3d](https://k3d.io/) >4.x.x
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)

## Agenda

* **[Prepare your local Kubernetes environment](docs/01_local_kubernetes.md)**
* **[Run an application on Kubernetes](docs/02_run_an_application.md)**
* **[Deploy a Wordpress](docs/03_wordpress.md)**

## Cleanup

Pretty simple we’ll drop the whole k3d cluster

```console
$ k3d cluster delete workshop
```