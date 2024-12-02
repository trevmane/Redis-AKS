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
## Connect to your database via redis-cli:

```bash
kubectl exec -it <pod-name> -- /bin/bash
```
```bash
redis-cli -h <private-ip-of-reaadb-service> -p <port>
```
