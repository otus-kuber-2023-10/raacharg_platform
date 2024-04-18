1. Запустил kind
2. Применил манифест [minio-statefulset.yaml](minio-statefulset.yaml)
3. Применил манифест [minio-headless-service.yaml](minio-headless-service.yaml)
4. Выполнил проверки - все ок.
5. Создал [minio-credentials-secret.yaml](minio-credentials-secret.yaml) со значениями в base64
6. Применил манифест [minio-credentials-secret.yaml](minio-credentials-secret.yaml)
7. Создание и использование PersistentVolumeClaim вPersistentVolumeClaim в Kubernetes(опционально)Kubernetes(опционально)
```
kubectl exec -it my-pod -- /bin/bash
root@my-pod:/# echo "12345" > /app/data/data.txt
root@my-pod:/# cat /app/data/data.txt 
12345
root@my-pod:/# exit
exit

kubectl delete pod my-pod 
pod "my-pod" deleted

kubectl apply -f my-pod-2.yaml 
pod/my-pod-2 created

kubectl exec -it my-pod-2 -- /bin/bash
root@my-pod-2:/# cat /app/data/data.txt 
12345
```