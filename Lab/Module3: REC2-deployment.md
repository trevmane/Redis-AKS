## Add AKS cluster to ~/.kube config:
```bash
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME
```
## Create Namespace in cluster:
```bash
kubectl create namespace $NS2
```
## Set context to that namespace:
```bash
kubectl config set-context --current --namespace=$NS2
```
## Query latest operator version:
```bash
VERSION=`curl --silent https://api.github.com/repos/RedisLabs/redis-enterprise-k8s-docs/releases/latest | grep tag_name | awk -F'"' '{print $4}'`
```
## Deploy operator with latest version:
```bash
kubectl apply -f https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/$VERSION/bundle.yaml
```
## Install haproxy-ingress controller:
```bash
helm install haproxy-ingress haproxy-ingress/haproxy-ingress --namespace $NS2
```

## Validate that our operator and ingress-controller deployments are up:
```bash
kubectl get deployments
```
## Validate that your ingress-controller service was created:
```bash
kubectl get svc
```

## Create Redis Enterprise Cluster (REC) custom resource:
```bash
kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1
kind: RedisEnterpriseCluster
metadata:
  name: $REC2
spec:
  nodes: 3
  redisEnterpriseNodeResources:
     limits:
       cpu: 2000m
       memory: 4Gi
     requests:
       cpu: 1000m
       memory: 2Gi
  ingressOrRouteSpec:
    apiFqdnUrl: api-${REC2}-${NS2}.${WILDCARD_DOMAIN2}
    dbFqdnSuffix: -db-${REC2}-${NS2}.${WILDCARD_DOMAIN2}
    ingressAnnotations:
      kubernetes.io/ingress.class: haproxy
      ingress.kubernetes.io/ssl-passthrough: "true"
    method: ingress
EOF
```
## Obtain externalIP address for ingress-controller service and add that to DNS record:
```bash
kubectl get svc <name-of-haproxy-ingress-svc>
```
## Obtain admission controller secret and save to env variable:
```bash 
unset CERT && CERT=`k get secret admission-tls -o jsonpath='{.data.cert}'` && echo $CERT
```

## replace placeholder namespace with our namespace and apply webhook.yaml file:
```bash
sed "s/OPERATOR_NAMESPACE/$NS2/g" webhook.yaml | k create -f -
```
## create a new webhook file for our 1st cluster:
```bash
cat > ${REC2}-webhook.yaml <<EOF
webhooks:
- name: redisenterprise.admission.redislabs
  clientConfig:
    caBundle: $CERT
  admissionReviewVersions: ["v1beta1"]
EOF
```
## Patch our validating webhook configuration with this new file:
```bash
kubectl patch ValidatingWebhookConfiguration redis-enterprise-admission --patch "$(cat ${REC2}-webhook.yaml)"
```
## Obtain our REC2 password and paste into secrets.yaml file under redis-enterprise-rc2 (you only need to paste the password):
```bash
kubectl get secret $REC2 -n redis -o jsonpath='{.data.password}'
```

```bash
apiVersion: v1
metadata:
  name: redis-enterprise-rc1
data:
  password: <my-password>
  username: ZGVtb0ByZWRpcy5jb20=
kind: Secret
type: Opaque
---
apiVersion: v1
metadata:
  name: redis-enterprise-rc2
data:
  password: <my-password>
  username: ZGVtb0ByZWRpcy5jb20=
kind: Secret
type: Opaque
```

## apply your secrets file to the current cluster (you will need to apply this secrets.yaml file to the other cluster as well):
```bash
kubectl apply -f secrets.yaml
```
