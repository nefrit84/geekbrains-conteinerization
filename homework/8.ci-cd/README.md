
## Подготовка
Проверяем доступность кластера

    $ kubectl cluster-info
    Kubernetes control plane is running at https://87.239.106.21:6443
    CoreDNS is running at https://87.239.106.21:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Кластер в строю, теперь настраиваем GitLab.

Генерируем SSH ключ командой `ssh-keygen` и добавляем его в настройки аккаунта **Preferences -> SSH keys**

Создаем новый проект с именем **geekbrains** и заливаем туда файлы из
учебного репозитория `https://github.com/adterskov/geekbrains-conteinerization/tree/master/practice/8.ci-cd`.

## Настраиваем интеграцию GitLab и Kubernetes
Отключаем **Shared Runners** (Settings -> CI/CD -> Runners)

Создаем Namespace для Runner'а

    $ kubectl create ns gitlab
    namespace/gitlab created

Добавляем в манифест Runner'а (репозиторий **geekbrains**) свой регистрационный токен
из `Settings -> CI/CD -> Runners -> Specific runners -> Set up a specific runner manually -> Registration token`

Применяем манифесты для Runner'а

    $ kubectl apply -n gitlab -f gitlab-runner.yaml
    serviceaccount/gitlab-runner created
    secret/gitlab-runner created
    configmap/gitlab-runner created
    role.rbac.authorization.k8s.io/gitlab-runner created
    rolebinding.rbac.authorization.k8s.io/gitlab-runner created
    deployment.apps/gitlab-runner created


Runner появился в списке **Available specific runners**

Создаем Namespace'ы для приложения

    $ kubectl create ns stage
    namespace/stage created

    $ kubectl create ns prod
    namespace/prod created

Создаем авторизационные объекты, чтобы Runner мог деплоить в наши Namespace'ы

    $ kubectl create sa deploy -n stage
    serviceaccount/deploy created

    $ kubectl create rolebinding deploy --serviceaccount stage:deploy --clusterrole edit -n stage
    rolebinding.rbac.authorization.k8s.io/deploy created

    $ kubectl create sa deploy -n prod
    serviceaccount/deploy created

    $ kubectl create rolebinding deploy --serviceaccount prod:deploy --clusterrole edit -n prod
    rolebinding.rbac.authorization.k8s.io/deploy created

Получаем токены для деплоя в Namespace'ы

    $ export NAMESPACE=stage; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}'
    ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklqVlNjRE54YkVWWWF6UlpNVmh5UTBWdlducFNORlZmV0cxcU56RjZkRmwzY21sTU5rSkVkMlJ3TldzaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUp6ZEdGblpTSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWamNtVjBMbTVoYldVaU9pSmtaWEJzYjNrdGRHOXJaVzR0TW0xa2FtUWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2laR1Z3Ykc5NUlpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WlhKMmFXTmxMV0ZqWTI5MWJuUXVkV2xrSWpvaU1qSmpPV013WW1RdFkyTTVaQzAwWVdObUxXRTBNbU10TUdVeVpURTFOVGN6TUdSbElpd2ljM1ZpSWpvaWMzbHpkR1Z0T25ObGNuWnBZMlZoWTJOdmRXNTBPbk4wWVdkbE9tUmxjR3h2ZVNKOS5zRjV2ZjBaV3VYc2tyVWZJQV9VQmJGRnNsaU5CZUVsbGt4M1RBbVRKZE1tNUIycjVQS0VvQ1FnOFFJMW1oNlMyQ3RoM1Nnb3pMb2FxbjJwV0hySWVQS2dTcUZZdV9wZ2d1WkllRkxEUkVLOEpBQWpwb3dQWjRrRG9iTml2dWNVbnY4M3ZSdlVVakNhbUNCQUZWSnBJZEhsR0hTakltQkpxRXZ6OTFreUttZmNZM080Wm9SbndCYTdyYV9RV010REk5aS1ib1R0RHk1QXJtVkJaUGhaTWxVc3NzUGhPWWd2NFZSQTdyS2xIMURiOEJVWGlNNTVJbGl5OWZlTVNsM2ZKS1BzQjJtZWppSFl5NUJKbWFvSk9pbEZwMDZfb3JoQ0VxMEFxV3VEdzRJVVdtUXFwSVJRaUhucVRGYllNSnZSZk9HWExiNm9Qb0VDb2RQQjdlUUZhQmc=

    $ export NAMESPACE=prod; kubectl get secret $(kubectl get sa deploy --namespace $NAMESPACE -o jsonpath='{.secrets[0].name}') --namespace $NAMESPACE -o jsonpath='{.data.token}'
    ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklqVlNjRE54YkVWWWF6UlpNVmh5UTBWdlducFNORlZmV0cxcU56RjZkRmwzY21sTU5rSkVkMlJ3TldzaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUp3Y205a0lpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkltUmxjR3h2ZVMxMGIydGxiaTA0Y2pReU1pSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMbTVoYldVaU9pSmtaWEJzYjNraUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lJeE5UWXhaVFUwTVMxa09UZGpMVFF3WTJZdFltVmtPUzAzWWpnd01ESTFPR0poTWpraUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZjSEp2WkRwa1pYQnNiM2tpZlEuZTNsTlc2RjBESHNiTXROcjZQMkRnNjJsVExxRFJJeUU4WFFvQU1vdmVpdmgzblRLVzI1VHVWckZuclNIWUhOVEdxaV9xSTMzMmFxOEE3QUxNUVgwcG1XYVNpeWJTLVB1OUNab0RSNEs2UjF2N1JBS2YwS2kyM29jWDBDTG04RGtDV3ptSjZnbmVPOXQxbkQ5dFVudVNDVTM0STRiVDA4SzB5Q2l0c0NqWFlyVXR2OU5pX1JaUUMzd1VhSWtOellDek1uMXRyeXV0UGxQcXk1TnM5b1E5alhxbzhmb0ZmTUNPcmQ5MGpodGxSYzk5amZIVkhGY3Q5THRvdVRSV3lYQzlrSVBhTzBXaFViTW9vSWR3Y2lremdDcFF2cERGSDBYNzNaOHllM1R2eldHdmh1ZkdUdFU0eXk4aDdjbnQ2NGw1bjNneEM1RkJ5Y2s3djZudnZINnRR

Помещаем токены в новые переменные проекта в Gitlab (Settings -> CI/CD -> Variables) с именами **K8S_STAGE_CI_TOKEN** и **K8S_PROD_CI_TOKEN**.

Создаем Token в **Settings -> Repository -> Deploy Tokens**. Из него нам понадобятся Username и Password.

Создаем секреты для авторизации Kubernetes в Gitlab registry. Используем для этого Username и Password из предыдущего шага.

    $ kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=gitlab+deploy-token-1629937 --docker-password=nYAffV-RbJqBfaAz3KhB --docker-email=admin@admin.admin -n stage
    secret/gitlab-registry created

    $ kubectl create secret docker-registry gitlab-registry --docker-server=registry.gitlab.com --docker-username=gitlab+deploy-token-1629937 --docker-password=nYAffV-RbJqBfaAz3KhB --docker-email=admin@admin.admin -n prod
    secret/gitlab-registry created

Патчим дефолтный сервис аккаунт для автоматического использования `pull secret`

    $ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n stage
    serviceaccount/default patched

    $ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlab-registry"}]}' -n prod
    serviceaccount/default patched


## Запуск приложения
Применяем манифесты для БД в `stage` и `prod`

    $ kubectl apply --namespace stage -f app/kube/postgres/
    secret/app created
    service/database created
    statefulset.apps/database created

    $ kubectl apply --namespace prod -f app/kube/postgres/
    secret/app created
    service/database created
    statefulset.apps/database created

Редактируем `app/kube/ingress.yaml` - для `host` прописываем значение `stage` и применяем манифесты для Namespace `stage`

    $ kubectl apply --namespace stage -f app/kube
    deployment.apps/geekbrains created
    ingress.networking.k8s.io/geekbrains created
    service/geekbrains created

Снова редактируем `app/kube/ingress.yaml` - для `host` прописываем значение `prod` и применяем манифесты для Namespace `prod`

    $ kubectl apply --namespace prod -f app/kube
    deployment.apps/geekbrains created
    ingress.networking.k8s.io/geekbrains created
    service/geekbrains created


## Проверяем работу приложения

Подсмотрим внешний IP нашего `ingres-controller (LoadBalancer)`.

    $ kubectl get svc -A
    NAMESPACE              NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
    default                kubernetes                           ClusterIP      10.254.0.1       <none>         443/TCP                      32h
    ingress-nginx          ingress-nginx-controller             LoadBalancer   10.254.243.89    87.239.110.8   80:30080/TCP,443:30443/TCP   32h
    ingress-nginx          ingress-nginx-controller-admission   ClusterIP      10.254.56.246    <none>         443/TCP                      32h
    ingress-nginx          ingress-nginx-controller-metrics     ClusterIP      10.254.52.205    <none>         9913/TCP                     32h
    ingress-nginx          ingress-nginx-default-backend        ClusterIP      10.254.140.129   <none>         80/TCP                       32h
    kube-system            calico-node                          ClusterIP      None             <none>         9091/TCP                     32h
    kube-system            calico-typha                         ClusterIP      10.254.24.216    <none>         5473/TCP                     32h
    kube-system            csi-cinder-controller-service        ClusterIP      10.254.55.111    <none>         12345/TCP                    32h
    kube-system            kube-dns                             ClusterIP      10.254.0.10      <none>         53/UDP,53/TCP,9153/TCP       32h
    kube-system            metrics-server                       ClusterIP      10.254.127.135   <none>         443/TCP                      32h
    kubernetes-dashboard   dashboard-metrics-scraper            ClusterIP      10.254.31.26     <none>         8000/TCP                     32h
    kubernetes-dashboard   kubernetes-dashboard                 ClusterIP      10.254.188.103   <none>         443/TCP                      32h
    opa-gatekeeper         gatekeeper-webhook-service           ClusterIP      10.254.47.61     <none>         443/TCP                      32h
    prod                   database                             ClusterIP      10.254.65.17     <none>         5432/TCP                     6m36s
    prod                   geekbrains                           ClusterIP      10.254.120.220   <none>         8000/TCP                     48s
    stage                  database                             ClusterIP      10.254.104.248   <none>         5432/TCP                     6m53s
    stage                  geekbrains                           ClusterIP      10.254.8.56      <none>         8000/TCP                     92s

EXTERNAL-IP 87.239.110.8, пробуем сделать к нему запрос на host stage

    $ curl 87.239.110.8/users -H "Host: stage" -X POST -d '{"name": "Vasiya", "age": 34, "city": "Vladivostok"}'
    <html>
    <head><title>502 Bad Gateway</title></head>
    <body>
    <center><h1>502 Bad Gateway</h1></center>
    <hr><center>nginx</center>
    </body>
    </html>

Данный вывод говорит о том, что приложение не стартонуло, так как не собралось в пайплайне. Пайплайн недоступен для новых юзверей из РФ.
Аккаунт GitLab невалидный.

ДЗ в сложившихся обстоятельствах считаю бессмысленным, так как нет возможности проверить верность принимаемых решений :((
