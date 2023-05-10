## 配置URL重定向的路由服务

Nginx会将路径完整转发到后端（如，从Ingress访问的/service1/api路径会直接转发到后端Pod的/service1/api/路径）。如果您后端的服务路径为/api，则会出现路径错误，导致404的情况。该情况下，您可以通过配置`rewrite-target`的方式，来将路径重写至需要的目录。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foo.bar.com
  namespace: default
  annotations:
    #URL重定向。
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
    # 在Ingress Controller的版本≥0.22.0之后，path中需要使用正则表达式定义路径，并在rewrite-target中结合捕获组一起使用。
      - path: /svc(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service: 
            name: web1-service
            port: 
              number: 80
```

访问

```shell
curl -k -H "Host: foo.bar.com"  http://<IP_ADDRESS>/svc/foo
web1: /foo
```

