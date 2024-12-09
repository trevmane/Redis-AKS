# Let's create a custom ingress resource so that we can access our UI service on a custom FQDN

## Switch to the cluster that contains our REC1

```bash
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name <AKS-cluster-for-REC1>
```

## Adjust the domain prefix and group-name in the command below before running:

```bash
k apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: haproxy
    ingress.kubernetes.io/ssl-passthrough: "true"
    redis.io/last-keys: '["ingress.kubernetes.io/ssl-passthrough", "kubernetes.io/ingress.class"]'
  labels:
    app: redis-enterprise
  name: ui
  namespace: redis
spec:
  rules:
  - host: <my-custom-domain-prefix>.<group-name>-1.demo.redislabs.com
    http:
      paths:
      - backend:
          service:
            name: rec1-ui
            port:
              name: ui
        path: /
        pathType: ImplementationSpecific
EOF
```

## Navigate to your custom UI FQDN that you created above in your browser

## Obtain the username and password for our REC1 cluster and use those to log in:

```bash
kubectl get secret rec1 -n redis -o jsonpath='{.data.username}' | base64 --decode
```
```bash
kubectl get secret rec1 -n redis -o jsonpath='{.data.password}' | base64 --decode
```
