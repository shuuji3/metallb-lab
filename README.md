# metallb-lab
🗼 metallb-lab: Experiments to publish two services using [MetalLB](https://metallb.universe.tf/) and [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) on [MicroK8s](https://microk8s.io/)

## Install microk8s and enable metallb and ingress addon

```shell
sudo snap install microk8s
sudo microk8s enable metallb ingress
```

## Apply configs

```shell
git clone git@github.com:shuuji3/metallb-lab.git && cd metallb-lab/
sudo microk8s kubectl apply -f ./
```

## Created resources

The above `kubectl apply` will create the following resources:

```shell
kubectl get svc,deploy,ingress
```

```
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    2d3h
service/web          ClusterIP   10.152.183.89    <none>        8080/TCP   52m
service/nginx        ClusterIP   10.152.183.194   <none>        80/TCP     26m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web     1/1     1            1           52m
deployment.apps/nginx   1/1     1            1           28m

NAME                         CLASS    HOSTS         ADDRESS     PORTS   AGE
ingress.extensions/ingress   <none>   example.com   127.0.0.1   80      27m
```

## Ingress

```shell
kubectl describe ingress
```

```
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
Name:             ingress
Namespace:        default
Address:          127.0.0.1
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host                               Path  Backends
  ----                               ----  --------
  example.com  
                                     /hello-app   web:8080   10.1.75.67:8080)
                                     /            nginx:80   10.1.205.4:80)
Annotations:                         nginx.ingress.kubernetes.io/rewrite-target: /$1
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  27m   nginx-ingress-controller  Ingress default/ingress
```

## Access services

You can now access two services with each path defined in the Ingress resource.

### svc/nginx at the path `http://<host-public-ip>/`

```shell
curl -s -H 'host: example.com' http://<host-public-ip>/ | head
```

should return

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
```

### svc/hello-app at the path `http://<host-public-ip>/hello-app/`

```shell
curl -s -H 'host: example.com' http://<host-public-ip>/hello-app
```

will return

```
Hello, world!
Version: 1.0.0
Hostname: web-79d88c97d6-sckw6
```
