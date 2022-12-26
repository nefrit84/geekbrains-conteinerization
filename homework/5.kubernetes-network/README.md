## Урок 5. Сетевые абстракции Kubernetes

Проверяем работоспособность кластера

    $ kubectl cluster-info
    Kubernetes control plane is running at https://87.239.110.8:6443
    CoreDNS is running at https://87.239.110.8:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Создаем namespace

    $ kubectl create ns redmine
    namespace/redmine created

Переключаем контекст на namespace redmine

    $ kubectl config set-context --current --namespace=redmine
    Context "default/kubernetes-cluster-5519" modified.

Готовим сетевой диск для базы

    $ kubectl apply -f pvc.yaml
    persistentvolumeclaim/pg-storage created

Проверяем его статус

    $ kubectl get pvc
    NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
    pg-storage   Bound    pvc-b3aa505c-63ba-48d9-a83e-a89ee8d5d742   10Gi       RWX            csi-ceph-ssd-gz1   65s

Подготавливаем `secret` для хранения пароля от базы

    $ kubectl create secret generic pg-secret --from-literal=PASS=rmdbpassword
    secret/pg-secret created

Разворачиваем базу

    $ kubectl apply -f deployment-pg.yaml
    deployment.apps/pg-db created

Разворачиваем сервис для доступа к базе из других контейнеров

    $ kubectl apply -f service-pg.yaml
    service/pg-service created

Подготавливаем `secret` для хранения ключа от Redmine

    $ kubectl create secret generic redmine-secret --from-literal=KEY=supersecretkey
    secret/redmine-secret created

Разворачиваем Redmine

    $ kubectl apply -f deployment-redmine.yaml
    deployment.apps/redmine-app created

Разворачиваем сервис для доступа к Redmine из других контейнеров

    $ kubectl apply -f service-redmine.yaml
    service/redmine-service created

Создаем `ingress`

    $ kubectl apply -f ingress.yaml
    ingress.networking.k8s.io/redmine-ingress created

Смотрим все ли поднялось

    $ kubectl get po
    NAME                          READY   STATUS    RESTARTS   AGE
    pg-db-754f8597b7-cmwwg        1/1     Running   0          33m
    redmine-app-6d9485d44-dzgn7   1/1     Running   0          29m

Ищем внешний IP для доступа к Redmine   

    $ kubectl get svc -A
    NAMESPACE              NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
    default                kubernetes                           ClusterIP      10.254.0.1       <none>          443/TCP                      17d
    ingress-nginx          ingress-nginx-controller             LoadBalancer   10.254.117.82    87.239.106.21   80:30080/TCP,443:30443/TCP   17d
    ingress-nginx          ingress-nginx-controller-admission   ClusterIP      10.254.120.182   <none>          443/TCP                      17d
    ingress-nginx          ingress-nginx-controller-metrics     ClusterIP      10.254.79.50     <none>          9913/TCP                     17d
    ingress-nginx          ingress-nginx-default-backend        ClusterIP      10.254.87.31     <none>          80/TCP                       17d
    kube-system            calico-node                          ClusterIP      None             <none>          9091/TCP                     17d
    kube-system            calico-typha                         ClusterIP      10.254.92.12     <none>          5473/TCP                     17d
    kube-system            csi-cinder-controller-service        ClusterIP      10.254.237.204   <none>          12345/TCP                    17d
    kube-system            kube-dns                             ClusterIP      10.254.0.10      <none>          53/UDP,53/TCP,9153/TCP       17d
    kube-system            metrics-server                       ClusterIP      10.254.139.11    <none>          443/TCP                      17d
    kubernetes-dashboard   dashboard-metrics-scraper            ClusterIP      10.254.48.96     <none>          8000/TCP                     17d
    kubernetes-dashboard   kubernetes-dashboard                 ClusterIP      10.254.189.191   <none>          443/TCP                      17d
    opa-gatekeeper         gatekeeper-webhook-service           ClusterIP      10.254.236.121   <none>          443/TCP                      17d
    redmine                pg-service                           ClusterIP      10.254.240.57    <none>          5432/TCP                     33m
    redmine                redmine-service                      ClusterIP      10.254.146.218   <none>          80/TCP                       29m

Внешний IP находится в `EXTERNAL-IP` у сервиса `nginx-ingress-controller` с типом `LoadBalancer`.

Открываем этот адрес в  браузере http://87.239.106.21

![скриншот](https://github.com/nefrit84/geekbrains-conteinerization/blob/hw5/homework/5.kubernetes-network/Screenshot-Redmine.png)
