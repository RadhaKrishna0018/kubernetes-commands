# kubernetes-commands
## kubectl
### Installation of kubernetes metrics server
```shell script
kubectl apply -f metrices.yaml
```
```shell script
kubectl apply -f https://raw.githubusercontent.com/RadhaKrishna0018/kubernetes-commands/master/metrices.yaml
```
### Installation of kubectl krew and kubectl kube-capacity plugins

# kubectl krew installation
```shell script
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

```shell script
# This command needs to be run as root user and should be run everytime you exit the user
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```
# kubectl krew installation of kube-capacity

```shell script
kubectl krew install resource-capacity
```
```shell script
kubectl resource-capacity
```
```shell script
kubectl resource-capacity --pods
```
```shell script
kubectl resource-capacity --pods --util
```
```shell script
kubectl resource-capacity --util --sort cpu.util
```


#### See all cluster nodes load (top)
```shell script
kubectl top nodes
```

#### Get cluster events
```shell script
# All cluster
kubectl get events

# Specific namespace events
kubectl get events --namespace=kube-system
```

#### Get all cluster nodes IPs and names
```shell script
# Single call to K8s API
kubectl get nodes -o json | grep -A 12 addresses

# A loop for more flexibility
for n in $(kubectl get nodes -o name); do \
  echo -e "\nNode ${n}"; \
  kubectl get ${n} -o json | grep -A 8 addresses; \
done
```

#### See all cluster nodes CPU and Memory requests and limits
```shell script
# With node names
kubectl describe nodes | grep -A 3 "Name:\|Resource .*Requests .*Limits" | grep -v "Roles:"

# Just the resources
kubectl describe nodes | grep -A 3 "Resource .*Requests .*Limits"
``` 

#### See all namespaces in a cluster
```shell script
# All pods in cluster
kubectl get namespace
```

#### See all cluster pods
```shell script
# All pods in cluster across all namespaces
kubectl get pods -A

# All pods in specific namespace
kubectl get pods -n kube-system

# All pods in specific namespace with more details
kubectl get pods -n kube-system -o wide
```

#### See all services
```shell script
# All services in cluster
kubectl get svc -A

# All services in specific namespace
kubectl get svc -n kube-system
```

#### See all cluster pods load (top)
```shell script
# All pods in cluster
kubectl top pods -A

# All pods in specific namespace
kubectl top pods -n kube-system
```
#### Get formatted list of container images in pods
Useful for listing all running containers in your cluster
```shell script
# Option 1 - just the container name
kubectl get pods -A -o jsonpath='{..containers[*].name}' | tr -s ' ' '\n'

# Option 2 - namespace, pod container images and tags
kubectl get pods -A -o=jsonpath='{range .items[*]}{.metadata.namespace},{.metadata.name},{.spec.containers[*].image}{"\n"}' | tr -s ' ' '\n'

# Option 3 - pod container images and tags
kubectl get pods -A -o=jsonpath='{..containers[*].image}' | tr -s ' ' '\n'
```

#### Get list of pods sorted by restart count
* Option 1 for all pods (Taken from [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources))
```shell script
kubectl get pods -A --sort-by='.status.containerStatuses[0].restartCount'
```

* Option 2 with a filter, and a CSV friendly output
```shell script
kubectl get pods -A | grep my-app | awk '{print $5 ", " $1 ", " $6}'  | sort -n -r
```

#### Get current replica count on all HPAs (Horizontal Pod Autoscaling)
```shell script
kubectl get hpa -A -o=custom-columns=NAME:.metadata.name,REPLICAS:.status.currentReplicas | sort -k2 -n -r
```

#### List non-running pods
```shell script
kubectl get pods -A --no-headers | grep -v Running | grep -v Completed
```

#### Top Pods by CPU or memory usage
```shell script
# Top 20 pods by highest CPU usage
kubectl top pods -A --sort-by=cpu | head -20

# Top 20 pods by highest memory usage
kubectl top pods -A --sort-by=memory | head -20

# Roll over all kubectl contexts and get top 20 CPU users
for a in $(kubectl ctx); do echo -e "\n---$a"; kubectl ctx $a; kubectl top pods -A --sort-by=cpu | head -20; done
```

#### Forward local port to a pod or service
```shell script
# Forward localhost port 8080 to a specific pod exposing port 8080
kubectl port-forward -n namespace1 web 8080:8080

# Forward localhost port 8080 to a specific web service exposing port 80
kubectl port-forward -n namespace1 svc/web 8080:80
```

#### Start a shell in a temporary pod
Note - Pod will terminate once exited
```shell script
# Ubuntu
kubectl run my-ubuntu --rm -i -t --restart=Never --image ubuntu -- bash

# CentOS
kubectl run my-centos --rm -i -t --restart=Never --image centos:8 -- bash

# Alpine
kubectl run my-alpine --rm -i -t --restart=Never --image alpine:3.10 -- sh

# Busybox
kubectl run my-busybox --rm -i -t --restart=Never --image busybox -- sh
```

### Rolling restarts
Roll a restart across all resources managed by a Deployment, DaemonSet or StatefulSet with **zero downtime**<br>
**IMPORTANT**: For a Deployment or StatefulSet, a zero downtime is possible only if initial replica count is **higher than 1**!
```shell script
# Deployment
kubectl -n <namespace> rollout restart deployment <deployment-name>

# DaemonSet
kubectl -n <namespace> rollout restart daemonset <daemonset-name>

# StatefulSet
kubectl -n <namespace> rollout restart statefulsets <statefulset-name>
```



### Resources
Most of the code above is self experimenting and reading the docs. Some are copied and modified to my needs from other resources...
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/
* https://medium.com/flant-com/kubectl-commands-and-tips-7b33de0c5476
* https://github.com/robscott/kube-capacity
* https://krew.sigs.k8s.io/docs/user-guide/setup/install/
