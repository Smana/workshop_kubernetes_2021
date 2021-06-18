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
* **[Deploy a MySQL database](docs/02_ecosystem.md)**
* **[Build your first chart](docs/03_build_chart.md)**
* **[Application lifecycle](docs/04_lifecycle.md)**
* **[Templating challenge](docs/05_templating_practice.md)**


## Cleanup

Pretty simple we’ll drop the whole k3d cluster

```console
$ k3d cluster delete workshop
```