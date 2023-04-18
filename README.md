# Kubernetes workshop

In this workshop, your task is to set up a local Kubernetes cluster, and then configure the resources needed to expose a web service.

## Prerequisites
An UNIX-environment/shell with Docker available (Git Bash seems to work)

### Kubectl and Kind
Kubectl is the CLI used to communicate with Kubernetes API servers. Kind provides a simple way of running Kubernetes on your local machine.
Follow the instructions here: https://kubernetes.io/docs/tasks/tools/

### Configure your cluster with an Ingress controller
The Kind cluster needs to be set up in a particular way for this:

```
cat <<EOF | kind create cluster -n workshop --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 8000
    protocol: TCP
  - containerPort: 443
    hostPort: 8443
    protocol: TCP
EOF
```

Then set Kubectl to use this cluster:
```
kubectl config use-context kind-workshop
```

Now, install the NGINX Ingress Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Use the following command to wait for this to be ready:
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

### K9s (nice to have)
A TUI (terminal user interface) for managing your Kubernetes clusters. Found here: https://k9scli.io/topics/install/

OpenLens is a GUI alternative: https://github.com/MuhammedKalkan/OpenLens#installation

## The Task

As mentioned above, your goal is to deploy a web service. The image can be found here:

```
ghcr.io/blixhavn/return-version:1.0.0
```

The script loadtest.sh can be used for continuously polling a web address. Use this together with updating the image version in the Deployment and apply it to the cluster, and then look at how the new version starts responding.

Use this to apply resources to the cluster:
```
kubectl apply -f <filename.yaml>
```

Verify that everything is configured correctly by visiting `localhost:8000` in your browser and see that you get a response.

### Scale

Horizontally scale your deployment, then use the `./loadtest.sh localhost:8000` script to inspect how requests are routed to different replicas.

### Rollout new version

While running the loadtest in a terminal, change the deployment to use version 2.0.0 of the image and apply it. If you have enough replicas you might see that version 1 and version 2 are living side by side for a little period.