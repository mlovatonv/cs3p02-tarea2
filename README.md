# Tarea 2

## Parte 1: Cluster Kubernetes

1. Crear un cluster kubernetes local con un único nodo (host, vm, containers).

Script

```bash
$ minikube start -p singlenode
```

Resultados

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.39.175:8443
KubeDNS is running at https://192.168.39.175:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
$ kubectl get nodes -owide
NAME         STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
singlenode   Ready    control-plane,master   3m47s   v1.20.2   192.168.39.175   <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
```

2. Crear un cluster kubernetes local multi-nodo (host, vm, containers). Por lo menos 2 workers.

Script

```bash
$ minikube start --nodes 2 -p multinode
```

Resultados

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.39.92:8443
KubeDNS is running at https://192.168.39.92:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
$ kubectl get nodes -owide
NAME            STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
multinode       Ready    control-plane,master   6m2s    v1.20.2   192.168.39.92    <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
multinode-m02   Ready    <none>                 4m44s   v1.20.2   192.168.39.142   <none>        Buildroot 2020.02.10   4.19.171         docker://20.10.3
```

## Parte 2: (Containerized) Application

1. Proponer y describir la aplicación: objetivo, arquitectura, funcionalidad, y otras características importantes de la aplicación.

Objetivo: Correr una aplicacion que funcione como un libro de visitas electronico

Arquitectura: MongoDB, PHP

Funcionalidad: Permite a visitantes dejar su opinion sobre la visita realizada a un sitio

## Parte 3: Aplication Deployment

1. Deployar la aplicación en los ambientes configurados: nodo único y multi-nodo. Verificar que la aplicación este usando 2 o más nodos.

### Parte 3.1 Deployando la aplicacion


#### MongoDB

```
kubectl apply -f application/guestbook/mongo-deployment.yaml
```

Verificar que el pod de mongodb esta corriendo

```
kubectl get pods -l app.kubernetes.io/name=mongo -l app.kubernetes.io/component=backend
```

```
kubectl apply -f application/guestbook/mongo-service.yaml
```

Verificar que el servicio de mongo esta corriendo

```
kubectl get service
```


#### Frontend

```
kubectl apply -f application/guestbook/frontend-deployment.yaml
```

Verificar que el pod de frontend esta corriendo

```
kubectl get pods -l app.kubernetes.io/name=guestbook -l app.kubernetes.io/component=frontend
```

```
kubectl apply -f application/guestbook/frontend-service.yaml
```

Verificar que el servicio de frontend esta corriendo
```
kubectl get services
```


Ver el servicio de frontend atraves de port-forward
```
kubectl port-forward svc/frontend 8080:80
```



2. Describir el flujo de la aplicación. Apoyarse creando uno o más flujos, que visualicen el ciclo de vida de la aplicación, y como interactúa con los componentes internos de Kubernetes. Presentar al menos 2 flujos, uno de alto nivel y otro de bajo nivel (o más detallado).


