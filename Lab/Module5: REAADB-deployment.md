## Apply reaadb.yaml to ONE cluster:

```bash
kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseActiveActiveDatabase
metadata:
  name: aadb1
spec:
  participatingClusters:
    - name: rc1
    - name: rc2
  globalConfigurations:
    replication: true
    shardCount: 1
EOF
```
## Connect to your database via redis-cli from within the cluster:

```bash
kubectl exec -it <pod-name> -- /bin/bash
```
```bash
redis-cli -h <clusterIP-of-reaadb-service> -p <port>
```
## Connect to your database via redis-cli from outside the cluster:

```bash
kubectl get ingress
```
We need to get our cluster certificate and save it to our local machine (or wherever you want to run redis-cli from)
```bash
kubectl exec -it rec1-0 -c redis-enterprise-node -- cat /etc/opt/redislabs/proxy_cert.pem > proxy_cert.pem
```

```bash
redis-cli -h <database-service-ingress-name> -p 443 --tls --cacert ./proxy_cert.pem
```

** Make sure you have at least version 7.0 of redis-cli for TLS support **
```bash
redis-cli --version
```
