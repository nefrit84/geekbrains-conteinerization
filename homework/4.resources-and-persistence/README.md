## Урок 4. Хранение данных и ресурсы

Используем кластер из домашнего задания №3. Проверяем его доступность

    $ kubectl cluster-info
    Kubernetes control plane is running at https://87.239.110.8:6443
    CoreDNS is running at https://87.239.110.8:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

Проверяем состояние нод на кластере    

    $ kubectl get node
    NAME                                STATUS   ROLES    AGE   VERSION
    kubernetes-cluster-5519-group-1-0   Ready    <none>   8d    v1.23.6
    kubernetes-cluster-5519-group-1-1   Ready    <none>   8d    v1.23.6
    kubernetes-cluster-5519-group-1-2   Ready    <none>   8d    v1.23.6
    kubernetes-cluster-5519-master-0    Ready    master   8d    v1.23.6

Создаем новый namespace pg

    $ kubectl create ns pg
    namespace/pg created

Переключаемся на созданный выше namespace pg

    $ kubectl config set-context --current --namespace=pg
    Context "default/kubernetes-cluster-5519" modified.

Создаем манифест для Persistent Volume и применяем его

    $ kubectl apply -f pvc.yaml
    persistentvolumeclaim/pg-storage created

Проверяем, что сетевой диск создался

    $ kubectl get pvc
    NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
    pg-storage   Bound    pvc-0f695773-8458-44bf-8285-4ab8bc7a9675   10Gi       RWX            csi-ceph-ssd-gz1   18s

Создаем предварительно secret для хранения пароля от БД.

    $ kubectl create secret generic pg-secret --from-literal=PASS=testpassword
    secret/pg-secret created

Проверим, что secret создался

    $ kubectl get secret pg-secret
    NAME        TYPE     DATA   AGE
    pg-secret   Opaque   1      68s

Проверим содержимое манифеста secret'а    

    $ kubectl get secret pg-secret -oyaml
    apiVersion: v1
    data:
      PASS: dGVzdHBhc3N3b3Jk
    kind: Secret
    metadata:
      creationTimestamp: "2022-12-05T20:46:13Z"
      name: pg-secret
      namespace: pg
      resourceVersion: "22957"
      selfLink: /api/v1/namespaces/pg/secrets/pg-secret
      uid: 46e0a17f-b270-4b50-b4c0-461b1b630217
    type: Opaque

Разворачиваем деплоймент с базой

    $ kubectl apply -f deployment.yaml
    deployment.apps/pg-db created

Проверим состояние PODа

    $ kubectl get pods
    NAME                    READY   STATUS    RESTARTS   AGE
    pg-db-7949d4bdc-rlmbf   1/1     Running   0          37s


## Проверка

Узнаем IP-адрес POD-а

    $ kubectl get pod -o wide
    NAME                    READY   STATUS    RESTARTS   AGE    IP              NODE                                NOMINATED NODE   READINESS GATES
    pg-db-7949d4bdc-rlmbf   1/1     Running   0          2m4s   10.100.79.192   kubernetes-cluster-5519-group-1-1   <none>           <none>

Запускаем POD, с которого будем тестировать POD с базой

    $ kubectl run -t -i --rm --image postgres:10.13 test bash
    If you don't see a command prompt, try pressing enter.
    root@test:/#

Подключаемся к БД

    root@test:/# psql -h 10.100.79.192 -U testuser testdatabase
    Password for user testuser:
    psql (10.13 (Debian 10.13-1.pgdg90+1))
    Type "help" for help.

    testdatabase=#

Создаем тестовую таблицу в базе

    testdatabase=# CREATE TABLE testtable (testcolumn VARCHAR (50) );
    CREATE TABLE

Проверяем что таблица создалась

    testdatabase=# \dt
               List of relations
     Schema |   Name    | Type  |  Owner
    --------+-----------+-------+----------
     public | testtable | table | testuser
    (1 row)

Выходим

    testdatabase=# \q
    root@test:/# exit

Удаляем POD с базой

    $ kubectl delete po pg-db-7949d4bdc-rlmbf
    pod "pg-db-7949d4bdc-rlmbf" deleted

Ждем пока POD пересоздастся, проверяем

    $ kubectl get po
    NAME                    READY   STATUS    RESTARTS   AGE
    pg-db-7949d4bdc-jsm27   1/1     Running   0          73s

Узнаем IP-адрес нового POD-а

    $ kubectl get pod -o wide
    NAME                    READY   STATUS    RESTARTS   AGE     IP              NODE                                NOMINATED NODE   READINESS GATES
    pg-db-7949d4bdc-jsm27   1/1     Running   0          2m29s   10.100.201.64   kubernetes-cluster-5519-group-1-2   <none>           <none>

Заходим в отдельный тестовый POD

    $ kubectl run -t -i --rm --image postgres:10.13 test bash
    If you don't see a command prompt, try pressing enter.
    root@test:/#

Подключаемся к БД

    root@test:/# psql -h 10.100.201.64 -U testuser testdatabase
    Password for user testuser:
    psql (10.13 (Debian 10.13-1.pgdg90+1))
    Type "help" for help.

Проверяем что таблица, созданная ранее осталась на месте

    testdatabase=# \dt
               List of relations
     Schema |   Name    | Type  |  Owner
    --------+-----------+-------+----------
     public | testtable | table | testuser
    (1 row)

Выходим

    testdatabase=# \q
    root@test:/# exit
