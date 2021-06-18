# Prepare your local Kubernetes environment

**Goal**: Having a running local Kubernetes environment

## Using k3d to create a cluster

In order to have an easily provisioned temporary playground we’ll make use of **k3d** which is a lightweight local Kubernetes instance.
(Note that they are alternatives to run a local Kubernetes cluster such as: `kubeadm`, `microk8s`, `minikube`)

After installing the binary you should enable the completion (bash or zsh) as follows (do the same for both helm and k3d).

```console
$ source <(k3d completion bash)
```

Then create the sandbox cluster named "**workshop**" with an additionnal worker

```console
$ k3d cluster create workshop -a 1
INFO[0000] Prep: Network
INFO[0000] Created network 'k3d-workshop' (ce74508d3fe09d8622f1ae83effd412d754dfdb441aa9d550723805f9b528c6b)
INFO[0000] Created volume 'k3d-workshop-images'
INFO[0001] Creating node 'k3d-workshop-server-0'
INFO[0001] Creating node 'k3d-workshop-agent-0'
INFO[0001] Creating LoadBalancer 'k3d-workshop-serverlb'
INFO[0001] Starting cluster 'workshop'
INFO[0001] Starting servers...
INFO[0001] Starting Node 'k3d-workshop-server-0'
INFO[0006] Starting agents...
INFO[0006] Starting Node 'k3d-workshop-agent-0'
INFO[0018] Starting helpers...
INFO[0018] Starting Node 'k3d-workshop-serverlb'
INFO[0019] (Optional) Trying to get IP of the docker host and inject it into the cluster as 'host.k3d.internal' for easy access
INFO[0023] Successfully added host record to /etc/hosts in 3/3 nodes and to the CoreDNS ConfigMap
INFO[0023] Cluster 'workshop' created successfully!
INFO[0023] --kubeconfig-update-default=false --> sets --kubeconfig-switch-context=false
INFO[0023] You can now use it like this:
kubectl config use-context k3d-workshop
kubectl cluster-info
```

As k3d is made to be used on top of docker you can see the status of the running containers. You should have 3 containers, one for the loadbalancing, one for the control-plane and an agent (worker).

```console
$ docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS              PORTS                             NAMES
4b5847b265dd   rancher/k3d-proxy:v4.4.6   "/bin/sh -c nginx-pr…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:43903->6443/tcp   k3d-workshop-serverlb
523a025087b3   rancher/k3s:v1.21.1-k3s1   "/bin/entrypoint.sh …"   About a minute ago   Up About a minute                                     k3d-workshop-agent-0
791b8a69bc1f   rancher/k3s:v1.21.1-k3s1   "/bin/entrypoint.sh …"   About a minute ago   Up About a minute                                     k3d-workshop-server-0
```

With `kubectl` you'll see 2 running pods: a control-plane and a worker

```console
$ kubectl get nodes
NAME                    STATUS   ROLES                  AGE     VERSION
k3d-workshop-agent-0    Ready    <none>                 2m34s   v1.21.1+k3s1
k3d-workshop-server-0   Ready    control-plane,master   2m44s   v1.21.1+k3s1
```

## The CLI configuration

The main interface to the Kubernetes API is `kubectl`. This CLI is configured with what we call a `kubeconfig`
you can have a look at its content wether by having a look at its default location is `~/.kube/config` or running the command

```console
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://0.0....
```

and you can check if the CLI is properly configured by running

```console
$ kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:43903
CoreDNS is running at https://0.0.0.0:43903/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:43903/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```


## kubectl plugins

It is really easy to extend the capabilities of he `kubectl` CLI.
Here is a basic "hello-world" example:

Write a dumb script, just ensure that it starts with `kubectl-` and put it in your path
```console
$ cat > kubectl-helloworld<<EOF
#!/bin/bash
echo "Hello world!"
EOF

$ chmod u+x && sudo mv kubectl-helloworld /usr/local/bin
```

Then it can be used as an argument of kubectl
```console
$ kubectl helloworld
Hello world!
```

You can find more information on how to create a kubectl plugin [here](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/)

In order to benefit from the plugins written by the community there's a tool named [**krew**](https://krew.sigs.k8s.io/)

First of all install the binary following this [doc](https://krew.sigs.k8s.io/docs/user-guide/setup/install/).

Update the local index

```console
$ kubectl krew update
Adding "default" plugin index from https://github.com/kubernetes-sigs/krew-index.git.
Updated the local copy of plugin index.
```

Browse the available plugins

```console
$ kubectl krew search
NAME                            DESCRIPTION                                         INSTALLED
access-matrix                   Show an RBAC access matrix for server resources     no
advise-psp                      Suggests PodSecurityPolicies for cluster.           no
allctx                          Run commands on contexts in your kubeconfig         no
apparmor-manager                Manage AppArmor profiles for cluster.               no
...
```

For the current workshop we'll make use of `ctx` `ns`

* ctx: Switch between contexts in your kubeconfig (Really helpful when you have multiple clusters to manage)
* ns: Switch between Kubernetes namespaces (Avoid to specify the namespace for each kubectl commands when working on a given namespace)

```console
$ kubectl krew install ctx ns
Updated the local copy of plugin index.
Installing plugin: ctx
...
```

Then you'll be able to switch between contexts (clusters) and namespaces.

```console
$ kubectl ns
Context "k3d-workshop" modified.
Active namespace is "kube-system".
```