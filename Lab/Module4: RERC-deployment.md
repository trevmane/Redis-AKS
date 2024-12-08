## Switch to cluster where secrets.yaml has NOT been applied:
```bash
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name <name-of-AKS-cluster>
```

## Change context to our REC namespace:
```bash
kubectl config set-context --current --namespace=redis
```

## Apply secrets.yaml file to remaining cluster to prepare for RERC deployment:
```bash
kubectl apply -f secrets.yaml
```

## Apply RERC.yaml on this cluster (you will need to also apply on the other cluster):
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

## Switch clusters and apply RERC to our other cluster:
```bash
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name <name-of-other-AKS-cluster>
```

```bash
kubectl config set-context --current --namespace=redis
```

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
