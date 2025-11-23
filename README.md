# Vault on AWS EKS Fargate Demo

This repository demonstrates deploying **HashiCorp Vault** on **AWS EKS Fargate** using `eksctl` and Kubernetes manifests.

## Prerequisites

- AWS CLI configured with appropriate IAM permissions
- `eksctl` installed
- `kubectl` installed
- Basic knowledge of Kubernetes and AWS EKS

---

## Step 1: Create EKS Fargate Cluster

Create an EKS cluster running entirely on Fargate:

```bash
eksctl create cluster \
  --name vault-fargate-demo \
  --region ap-south-1 \
  --fargate
```

## Step 2: Update your kubeconfig to connect to the new cluster:

```bash
aws eks update-kubeconfig --region ap-south-1 --name vault-fargate-demo

```
## Step 3: kubectl create namespace vault / Create a Fargate profile for the Vault namespace:
```bash
kubectl create namespace vault

eksctl create fargateprofile \
  --cluster vault-fargate-demo \
  --name vault-profile \
  --namespace vault


```

## Step 4: create handson.yaml

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault
  namespace: vault
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vault
  template:
    metadata:
      labels:
        app: vault
    spec:
      containers:
        - name: vault
          image: hashicorp/vault:1.21.1
          ports:
            - containerPort: 8200
          env:
            - name: VAULT_DEV_ROOT_TOKEN_ID
              value: "root-token-demo"
          args:
            - "server"
            - "-dev"
            - "-dev-listen-address=0.0.0.0:8200"
---
apiVersion: v1
kind: Service
metadata:
  name: vault
  namespace: vault
spec:
  type: ClusterIP
  selector:
    app: vault
  ports:
    - port: 8200
      targetPort: 8200
      protocol: TCP

```

## Step 5: final steps

```bash
kubectl apply -f handson.yaml

kubectl port-forward -n vault svc/vault 8200:8200

export VAULT_ADDR=http://127.0.0.1:8200

eksctl delete cluster --name vault-fargate-demo --region ap-south-1


```

