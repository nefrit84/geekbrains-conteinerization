## Урок 7. Продвинутые абстракции

Проверяем работоспособность кластера

    $ kubectl cluster-info
    Kubernetes control plane is running at https://87.239.106.21:6443
    CoreDNS is running at https://87.239.106.21:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


Разворачиваем абстракции

    $ kubectl apply -f configmap.yaml
    configmap/prometheus-config created

    $ kubectl apply -f serviceaccount.yaml
    serviceaccount/prometheus created

    $ kubectl apply -f clusterrole.yaml
    clusterrole.rbac.authorization.k8s.io/prometheus created

    $ kubectl apply -f clusterrolebinding.yaml
    clusterrolebinding.rbac.authorization.k8s.io/prometheus created

    $ kubectl apply -f statefulset.yaml
    statefulset.apps/prometheus created

    $ kubectl apply -f service.yaml
    service/prometheus created

    $ kubectl apply -f ingress.yaml
    ingress.extensions/prometheus created

    $ kubectl create -f daemonset.yaml
    Warning: spec.template.spec.nodeSelector[beta.kubernetes.io/os]: deprecated since v1.14; use "kubernetes.io/os" instead
    daemonset.apps/node-exporter created

Проверяем POD'ы

    $ kubectl get po
    NAME                  READY   STATUS    RESTARTS   AGE
    node-exporter-4nhhn   1/1     Running   1          3h23m
    node-exporter-jmzr9   1/1     Running   1          3h23m
    node-exporter-q9xjv   1/1     Running   1          3h23m
    node-exporter-szrcf   1/1     Running   1          3h23m
    prometheus-0          1/1     Running   1          3h26m

Ищем внешний IP для доступа к веб-интерфейсу **Prometheus**

    $ kubectl get svc -A
    NAMESPACE              NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
    default                kubernetes                           ClusterIP      10.254.0.1       <none>         443/TCP                      3h43m
    default                prometheus                           ClusterIP      10.254.240.62    <none>         80/TCP                       3h2m
    ingress-nginx          ingress-nginx-controller             LoadBalancer   10.254.243.89    87.239.110.8   80:30080/TCP,443:30443/TCP   3h41m
    ingress-nginx          ingress-nginx-controller-admission   ClusterIP      10.254.56.246    <none>         443/TCP                      3h41m
    ingress-nginx          ingress-nginx-controller-metrics     ClusterIP      10.254.52.205    <none>         9913/TCP                     3h41m
    ingress-nginx          ingress-nginx-default-backend        ClusterIP      10.254.140.129   <none>         80/TCP                       3h41m
    kube-system            calico-node                          ClusterIP      None             <none>         9091/TCP                     3h42m
    kube-system            calico-typha                         ClusterIP      10.254.24.216    <none>         5473/TCP                     3h42m
    kube-system            csi-cinder-controller-service        ClusterIP      10.254.55.111    <none>         12345/TCP                    3h42m
    kube-system            kube-dns                             ClusterIP      10.254.0.10      <none>         53/UDP,53/TCP,9153/TCP       3h42m
    kube-system            metrics-server                       ClusterIP      10.254.127.135   <none>         443/TCP                      3h41m
    kubernetes-dashboard   dashboard-metrics-scraper            ClusterIP      10.254.31.26     <none>         8000/TCP                     3h42m
    kubernetes-dashboard   kubernetes-dashboard                 ClusterIP      10.254.188.103   <none>         443/TCP                      3h42m
    opa-gatekeeper         gatekeeper-webhook-service           ClusterIP      10.254.47.61     <none>         443/TCP                      3h40m

Открываем в браузере адрес `87.239.110.8`


![скриншот](https://github.com/nefrit84/geekbrains-conteinerization/blob/HW7/homework/7.advanced-abstractions/prometheus-app.png)

Согласно заданию открываем `Status -> Targets`, видим все ноды кластера.

![скриншот](https://github.com/nefrit84/geekbrains-conteinerization/blob/HW7/homework/7.advanced-abstractions/prometheus-status-targets.png)

На вкладке Graph выполняем запрос `node_load1` - это минутный Load Average для каждой из нод в кластере.

![скриншот](https://github.com/nefrit84/geekbrains-conteinerization/blob/HW7/homework/7.advanced-abstractions/prometheus-node-load.png)
