# Service Definition

- Let us create a yaml file for service

```
vi nginx-service.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
 ```

 - Let us create the service now

 ```
 kubectl create -f nginx-service.yaml
 ```

 