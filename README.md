# Redis-AKS Deployment Instructions:


## Prerequisites
Make sure you have the following tools installed:
- **Azure CLI** (`az`)
- **kubectl**
- **helm**

## Add an alias for kubectl to ``` ~/.zshrc``` or ```~/.bashrc```
```bash
alias k='kubectl'
```

## Step 1: Login to Your Azure Account
Use the following command to log in:

```bash
az login
```

Create your environment variables as follows:

```bash
export REC1=rec1
export NS1=redis
export REC2=rec2
export NS2=redis
export WILDCARD_DOMAIN1=<groupname>-1.demo.redislabs.com
export WILDCARD_DOMAIN2=<groupname>-2.demo.redislabs.com
```

## Lab Modules Outline:
1) Prerequisites for deployment
2) Deploy RS Operator, ingress and REC in first region
3) Deploy RS operator, ingress and REC in second region
4) Apply REC secrets to each cluster and deploy RERC custom resource
5) Deploy REAADB custom resource
