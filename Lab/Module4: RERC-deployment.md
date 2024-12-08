## Change context to our REC namespace:
```bash
kubectl config set-context --current --namespace=redis
```

## Apply secrets.yaml file to BOTH clusters to prepare each cluster for RERC deployment:
```bash
kubectl apply -f <name-of-secrets-file>.yaml
```

## Apply RERC.yaml on ONE cluster (you don't need to apply on both):
```bash
kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseRemoteCluster
metadata:
  name: rc1
spec:
  recName: ${REC1}
  recNamespace: ${NS1}
  apiFqdnUrl: api-${REC1}-${NS1}.${WILDCARD_DOMAIN1}
  dbFqdnSuffix: -db-${REC1}-${NS1}.${WILDCARD_DOMAIN1}
  secretName: redis-enterprise-rc1
---
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseRemoteCluster
metadata:
  name: rc2
spec:
  recName: ${REC2}
  recNamespace: ${NS2}
  apiFqdnUrl: api-${REC2}-${NS2}.${WILDCARD_DOMAIN2}
  dbFqdnSuffix: -db-${REC2}-${NS2}.${WILDCARD_DOMAIN2}
  secretName: redis-enterprise-rc2
EOF
```
## Confirm that the RERC was deployed successfully:
```bash
kubectl get rerc
```
