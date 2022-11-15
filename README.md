1) Устанавливаем  Gatekeeper в кластер Kubernetes, предварительно создав namespace "gatekeeper-system":

```bash
kubectl create namespace gatekeeper-system

helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace
```

2) Проверяем, что все установлено и запущено

```bash
kubectl get all -n gatekeeper-system
```

3) Проверяем validation webhook от Gatekeeper:

```bash
kubectl get validatingwebhookconfigurations
```

4) Попробуем запустить pod, который изначально является нежелательным в системе и имеет дыру в безопасности. Этот pod будет запускаться на controleplane-ноде и монтировать внутрь себя весь корневой каталог в папку /host.

```bash
cd ./check_privileges/
kubectl create -f bad_pod.yaml
```

5) Убедимся, что Pod запустился. Попробуем зайти внутрь и посмотреть на файлы хостовой системы

```bash
kubectl exec bad-pod -it -- bash
ls -l /host/
```

6) Такое поведение недопустимо на продакшн с точки зрения безопасности, поэтому запретим запуск подобных pod'ов.
Для начала удалим текущий pod

```bash
kubectl delete pod bad-pod
```

7) Применяем настройки Open Policy Agent.
В файле template.yaml у нас хранится описание самого правила, а в файле constraint.yaml - значения для этого правила.

```bash
kubectl create -f template.yaml
kubectl create -f constraint.yaml
```
8) Попробуем снова запустить "плохой" pod:

```bash
kubectl create -f bad_pod.yaml
```

9) Видим ошибку со стороны GateKeeper на запрещенные поля в манифесте pod'а. Стоит проверить что своими действиями мы не сломали создание "хороших" pod'ов, т.е. запуск безопасных манифестов в кластере все еще возможен:

```bash
kubectl create -f good_pod.yaml
```

