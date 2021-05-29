# Tarea 2

## Parte 1: Cluster Kubernetes

1. Crear un cluster kubernetes local con un único nodo (host, vm, containers).

#### Cluster command

```bash
minikube start -p singlenode
```

#### Cluster info

```console
foo@bar:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.39.175:8443
KubeDNS is running at https://192.168.39.175:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
foo@bar:~$ kubectl get nodes -o wide
NAME         STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
singlenode   Ready    control-plane,master   3m47s   v1.20.2   192.168.39.175   <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
```

2. Crear un cluster kubernetes local multi-nodo (host, vm, containers). Por lo menos 2 workers.

#### Cluster command

```bash
minikube start --nodes 2 -p multinode
```

#### Cluster info

```console
foo@bar:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.39.92:8443
KubeDNS is running at https://192.168.39.92:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
foo@bar:~$ kubectl get nodes -owide
NAME            STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
multinode       Ready    control-plane,master   6m2s    v1.20.2   192.168.39.92    <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
multinode-m02   Ready    <none>                 4m44s   v1.20.2   192.168.39.142   <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
```

## Parte 2: (Containerized) Application

1. Proponer y describir la aplicación: objetivo, arquitectura, funcionalidad, y otras características importantes de la aplicación.

#### Objetivo

Correr una aplicacion que funcione como un libro de visitas electronico

#### Arquitectura

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│              ├─────►              ├─────►              ├─────►              │
│  Web Client  │     │  Web Server  │     │  PHP Module  │     │   MongoDB    │
│              ◄─────┤              ◄─────┤              ◄─────┤              │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

#### Funcionalidad

Permite a visitantes dejar su opinion sobre la visita realizada a un sitio

## Parte 3: Application Deployment

1. Deployar la aplicación en los ambientes configurados: nodo único y multi-nodo. Verificar que la aplicación este usando 2 o más nodos.

#### App configuration

##### MongoDB

```bash
kubectl apply -f app/deployments/mongo.yaml
kubectl apply -f app/services/mongo.yaml
```

##### Frontend

```bash
kubectl apply -f app/deployments/frontend.yaml
kubectl apply -f app/services/frontend.yaml
```

##### Port forwarding

```bash
kubectl port-forward svc/frontend 8080:80
```

#### Single-node cluster

Por defecto, minikube creara un cluster de 1 nodo.

```bash
minkube start
```

##### Pods info

```console
foo@bar:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
frontend-848d88c7c-2pqxv   1/1     Running   0          70m   172.17.0.7   minikube   <none>           <none>
frontend-848d88c7c-hxkbx   1/1     Running   0          70m   172.17.0.6   minikube   <none>           <none>
frontend-848d88c7c-xhprj   1/1     Running   0          70m   172.17.0.8   minikube   <none>           <none>
mongo-75f59d57f4-q74dt     1/1     Running   0          74m   172.17.0.5   minikube   <none>           <none>
```

#### Multi-node cluster

```bash
kubectl node add
```

Ahora tenemos 2 nodos...

```console
foo@bar:~$ kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
minikube       Ready    control-plane,master   53m   v1.20.2
minikube-m02   Ready    <none>                 15m   v1.20.2
```

Para probar nuestro cluster, hay que replicar algun pod (por ejemplo, el de frontend), asi nuestro load balancer distribuira algunas de esas replicas a nuestro nuevo nodo.

```bash
kubectl scale deployment frontend --replicas=5
```

##### Pods info

```console
foo@bar:~$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
frontend-848d88c7c-fcnfp   1/1     Running   0          11m   172.17.0.3   minikube-m02   <none>           <none>
frontend-848d88c7c-nkf2j   1/1     Running   0          53m   172.17.0.5   minikube       <none>           <none>
frontend-848d88c7c-tf4wg   1/1     Running   0          11m   172.17.0.2   minikube-m02   <none>           <none>
frontend-848d88c7c-x2dnm   1/1     Running   0          53m   172.17.0.4   minikube       <none>           <none>
frontend-848d88c7c-zgj6b   1/1     Running   0          53m   172.17.0.6   minikube       <none>           <none>
mongo-75f59d57f4-kwxm6     1/1     Running   0          55m   172.17.0.3   minikube       <none>           <none>
```

Veremos que nuestras nuevas dos replicas han sido instanciadas en el segundo nodo.

2. Describir el flujo de la aplicación. Apoyarse creando uno o más flujos, que visualicen el ciclo de vida de la aplicación, y como interactúa con los componentes internos de Kubernetes. Presentar al menos 2 flujos, uno de alto nivel y otro de bajo nivel (o más detallado).

## Referencias

- [Install Tools](https://kubernetes.io/docs/tasks/tools/)
- [Example: Deploying PHP Guestbook application with MongoDB](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)
