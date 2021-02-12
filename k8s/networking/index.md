# Ingress

Example of an ingress yaml file

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-networking
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```