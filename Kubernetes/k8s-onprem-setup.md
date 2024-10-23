## jackdaniels' on prem cluster setup

- ref: https://github.com/jdmedeiros/k8s.git || https://github.com/rlcosta177/k8s-amazlinux2-jdaniels/tree/original_v126_update (forked)
- branch: original_v126_update

## Update instance and install the development tools
```bash
sudo yum update -y
sudo yum group install 'Development Tools' -y
```

2.git clone do repo do prof
```bash
git clone https://github.com/jdmedeiros/k8s.git
cd k8s
git checkout original_v126_update
git pull
```

3.(OPTIONAL) comment out the aws stuff in 0-install-kubernetes-tools | Only necessary if you've done step 3 beforehand
```bash
sudo yum remove awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

4. Troubleshooting
- sudo nano /etc/profile -> add :/usr/local/bin
  - (adds a shortcut to any package in /usr/local/bin | ex: instead of doing '/usr/local/bin/go', you can now just use 'go')

## Important kubectl commands(kubernetes)

```bash
kubectl get all
kubectl delete(deployment & service/flask-nodeport(na tabela))
kubectl get pods -o=jsonpath="{range .items[*]}{.status.podIP}{','}{end}"
kubectl delete all --all --all-namespaces (dps tens que correr o ./13-... do professor)
k rollout restart deployment coredns -n kube-system

kubectl create: Used to create resources like pods, deployments, services, and more.
Example: kubectl create deployment my-deployment --image=my-image

kubectl apply: Applies a configuration to a resource or set of resources by filename, stdin, or URL.
Example: kubectl apply -f my-manifest.yaml

kubectl get: Retrieves one or more resources. This is one of the most commonly used commands.
Example: kubectl get pods

kubectl describe: Provides detailed information about a specific resource or group of resources.
Example: kubectl describe pod my-pod

kubectl delete: Deletes one or more resources.
Example: kubectl delete pod my-pod

kubectl logs: Retrieves the logs of a container in a pod.
Example: kubectl logs my-pod

kubectl exec: Executes a command in a container within a pod.
Example: kubectl exec -it my-pod -- /bin/bash

kubectl scale: Changes the number of replicas of a deployment, replication controller, or replica set.
Example: kubectl scale deployment/my-deployment --replicas=3

kubectl rollout: Manages the rollout of updates to a deployment or rollout history.
Example: kubectl rollout status deployment/my-deployment

kubectl port-forward: Forwards one or more local ports to a pod.
Example: kubectl port-forward my-pod 8080:80
```
