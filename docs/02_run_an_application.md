
# Run an application on Kubernetes

## Namespaces

Namespaces allow to logically distribute your applications, generally based on teams, projects or applications stacks.
Resources names are unique within a namespace that means that you could have a service named `webserver` on 2 different namespaces.

They can be used to isolate applications using network policies, or to define quotas.

For this training we'll work on a namespace named `foo`

```console
$ kubectl create ns foo
namespace/foo created
```

And we'll make use of the plugin installed in the previous section to set the default namespace as follows

```console
$ kubectl ns
Context "k3d-workshop" modified.
Active namespace is "foo".
```

Check that your `kubectl` is properly configured, here you can see the cluter and the namespace:

```console
$ kubectl config get-contexts
CURRENT   NAME           CLUSTER        AUTHINFO             NAMESPACE
*         k3d-workshop   k3d-workshop   admin@k3d-workshop   foo
```

## Create your first pod

Creating resources in Kubernetes is often done by applying yaml content through the API.

Start by creating a pretty simple pod:

```console
$ kubectl apply -f manifests/pod.yaml --namespace foo
pod/web created

$ kubectl get po
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          98s
```

We can get detailed information about the pod as follows

```console
$ kubectl describe po web -l run=web
Name:         web
Namespace:    foo
Priority:     0
Node:         k3d-workshop-agent-0/172.20.0.3
Start Time:   Fri, 18 Jun 2021 17:05:46 +0200
Labels:       run=web
Annotations:  <none>
Status:       Running
IP:           10.42.1.6
...
```

Or even get a specific attribute, here is an example to get the pod's IP

```console
$ kubectl get po web --template={{.status.podIP}}
10.42.1.6
```

This is worth noting that a pod isn't controlled by a `replicaset`. That means that when it is deleted, it is not restarted automatically.

```console
$ kubectl delete po web
pod "web" deleted

$ kubectl get po
No resources found in foo namespace.
```

## Create a simple webserver deployment

kubectl create deployment podinfo --image stefanprodan/podinfo
deployment.apps/podinfo created

kubectl get deploy
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
podinfo   1/1     1            1           14s

kubectl get replicasets
NAME                 DESIRED   CURRENT   READY   AGE
podinfo-7fbb45ccfc   1         1         1       36s

kubectl scale deploy podinfo --replicas 6
deployment.apps/podinfo scaled

kubectl rollout status deployment podinfo
Waiting for deployment "podinfo" rollout to finish: 4 of 6 updated replicas are available...
Waiting for deployment "podinfo" rollout to finish: 5 of 6 updated replicas are available...
deployment "podinfo" successfully rolled out

kubectl get po -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP           NODE                    NOMINATED NODE   READINESS GATES
podinfo-7fbb45ccfc-dwxtx   1/1     Running   0          114s   10.42.1.8    k3d-workshop-agent-0    <none>           <none>
podinfo-7fbb45ccfc-p2djv   1/1     Running   0          34s    10.42.1.11   k3d-workshop-agent-0    <none>           <none>
podinfo-7fbb45ccfc-4fk9z   1/1     Running   0          34s    10.42.1.9    k3d-workshop-agent-0    <none>           <none>
podinfo-7fbb45ccfc-gqwz6   1/1     Running   0          34s    10.42.1.10   k3d-workshop-agent-0    <none>           <none>
podinfo-7fbb45ccfc-4qgvs   1/1     Running   0          34s    10.42.0.8    k3d-workshop-server-0   <none>           <none>
podinfo-7fbb45ccfc-r6dn5   1/1     Running   0          34s    10.42.0.9    k3d-workshop-server-0   <none>           <none>

kubectl delete po podinfo-7fbb45ccfc-r6dn5
pod "podinfo-7fbb45ccfc-r6dn5" deleted

kubectl get deploy
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
podinfo   6/6     6            6           18m

kubectl set image deployment podinfo podinfo=stefanprodan/podinfo:5.2.1
deployment.apps/podinfo image updated


kubectl get rs -o wide
NAME                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                       SELECTOR
podinfo-7fbb45ccfc   0         0         0       21m   podinfo      stefanprodan/podinfo         app=podinfo,pod-template-hash=7fbb45ccfc
podinfo-564b4ddd7c   6         6         6       30s   podinfo      stefanprodan/podinfo:5.2.1   app=podinfo,pod-template-hash=564b4ddd7c

kubectl rollout undo deployment podinfo
deployment.apps/podinfo rolled back

kubectl get rs -o wide
NAME                 DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                       SELECTOR
podinfo-7fbb45ccfc   6         6         6       22m   podinfo      stefanprodan/podinfo         app=podinfo,pod-template-hash=7fbb45ccfc
podinfo-564b4ddd7c   0         0         0       77s   podinfo      stefanprodan/podinfo:5.2.1   app=podinfo,pod-template-hash=564b4ddd7c

## Expose a deployment

kubectl expose deploy podinfo --port 9898
service/podinfo exposed

$ kubectl get svc -o yaml podinfo

```console
apiVersion: v1
kind: Service
metadata:
  labels:
    app: podinfo
  name: podinfo
  namespace: foo
spec:
  clusterIP: 10.43.47.17
  clusterIPs:
  - 10.43.47.17
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 9898
  selector:
    app: podinfo
```

kubectl get endpoints
NAME      ENDPOINTS                                                     AGE
podinfo   10.42.0.16:9898,10.42.0.17:9898,10.42.1.18:9898 + 3 more...   92s

curl http://localhost:9898
Handling connection for 9898
{
  "hostname": "podinfo-7fbb45ccfc-sbqht",
  "version": "6.0.0",
  "revision": "",
  "color": "#34577c",
  "logo": "https://raw.githubusercontent.com/stefanprodan/podinfo/gh-pages/cuddle_clap.gif",
  "message": "greetings from podinfo v6.0.0",
  "goos": "linux",
  "goarch": "amd64",
  "runtime": "go1.16.5",
  "num_goroutine": "6",
  "num_cpu": "16"
}
