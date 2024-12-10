# EKS Auto-Mode Demo
source: https://aws.amazon.com/blogs/containers/getting-started-with-amazon-eks-auto-mode/

## Creating the Cluster & Deploying the App

1. Creating the EKS Cluster (**cluster.yaml**)
```
eksctl create cluster -f cluster.yaml
```

2. Check the cluster State

```
kubectl get nodes,pods,deploy
```

3. Creating an Ingress (**ingress.yaml**)

```
kubectl apply -f ingress.yaml
```

4. Creating the Storage Class (**ebs-sc.yaml**)
```
kubectl apply -f ebs-sc.yaml
```

5. Using Helm to provision the application (retail.store)

```
helm install -f values.yaml retail-store-app oci://public.ecr.aws/aws-containers/retail-store-sample-chart --version 0.8.3
```

6. Waiting for the nodes to be ready (Karpenter)

```
kubectl wait --for=condition=Ready nodes --all
```

7. Inspecting the Statefulset

```
kubectl get statefulset retail-store-app-catalog-mysql \
  -o jsonpath='{.spec.volumeClaimTemplates}' | jq .
```

8. Get the Ingress URL

```
kubectl get ingress retail-store-app-ui -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}"
```

## Scaling the App 

1. Deploying the Metrics Server

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

2. Getting the Horizontal Pod Autoscaler

```
kubectl get hpa retail-store-app-ui
```

3. Generating Load
```
kubectl run load-generator \
  --image=public.ecr.aws/amazonlinux/amazonlinux:2023 \
  --restart=Never -- /bin/sh -c "while sleep 0.01; do curl http://retail-store-app-ui/home; done"
```

4. Watching the Nodes

```
kubectl get nodes --watch
```

5. Deleting the Load Generator
```
kubectl delete pod load-generator
```

## CleanUp

```
kubectl delete deploy -n kube-system metrics-server 

helm uninstall retail-store-app

kubectl delete pvc/data-retail-store-app-catalog-mysql-0

eksctl delete cluster --name eks-auto-mode-demo
```