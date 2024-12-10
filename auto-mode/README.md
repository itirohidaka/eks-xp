# EKS Auto-Mode Demo
source: https://aws.amazon.com/blogs/containers/getting-started-with-amazon-eks-auto-mode/


1. Creating the EKS Cluster (**cluster.yaml**)
```
eksctl create cluster -f cluster.yaml
```

1. Check the cluster State

```
kubectl get nodes
kubectl get pods
```

1. Creating an Ingress (**ingress.yaml**)

```
kubectl apply -f ingress.yaml
```

1. Creating the Storage Class (**ebs-sc.yaml**)
```
kubectl apply -f ebs-sc.yaml
```

1. Using Helm to provision the application (retail.store)

```
helm install -f values.yaml retail-store-app oci://public.ecr.aws/aws-containers/retail-store-sample-chart --version 0.8.3
```

1. Waiting for the nodes to be ready (Karpenter)

```
kubectl wait --for=condition=Ready nodes --all
```

1. Inspecting the Statefulset

```
kubectl get statefulset retail-store-app-catalog-mysql \
  -o jsonpath='{.spec.volumeClaimTemplates}' | jq .
```
