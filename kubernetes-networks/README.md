В выполнении задания дошел до пункта
```
Установка MetalLB
MetalLB позволяет запустить внутри кластера L4-балансировщик,
который будет принимать извне запросы к сервисам и раскидывать их
между подами. Установка его проста:
� В продуктиве так делать не надо. Сначала стоит скачать файл и
разобраться, что там внутри
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.0/manifests
/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-
literal=secretkey="$(openssl rand -base64 128)"
```
Ссылка на https://raw.githubusercontent.com/metallb/metallb/v0.13.0/manifests/metallb.yaml не работает
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.0/manifests/metallb.yaml
error: unable to read URL "https://raw.githubusercontent.com/metallb/metallb/v0.13.0/manifests/metallb.yaml", server reported 404 Not Found, status code=404
```
Соответственно дальше задание не выполнял.