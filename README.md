# Using kubectl
- kubectl cluster-info --context kind-kind
    - context comes from ~/.kube/config
-


# Using Kind
Kind consists of:
    - Go packages implementing cluster creation, image build, etc.
    - A CLI `kind` built on those packages.
    - Docker image(s) (kindest/node) written to run systemd, Kubernetes, etc.
    - `kubetest`

- `kind create cluster`
    - bootstraps a Kubernetes cluster using the kindest/node pre-built node image (can use --image to specify a different image).
    - `--name` to name the cluster (default 'kind')
    - `--wait 5m` to wait until control plane is ready
    - docker ps will show a `kind-control-plane` image
    - `--config config.yaml` can specify a config (see: https://kind.sigs.k8s.io/docs/user/quick-start/)

- Once created, use `kubectl` to interact with it
    - The Kind config file is stored in ~/.kube/config or path in $KUBECONFIG if set.

- kind get clusters
- kubectl cluster-info --context kind-kind
    (context is always prefixed with kind-)

- kind delete cluster --name kind

## Loading images into the cluster
- docker build -t dummyapp:develop.latest .
- kind load docker-image dummyapp:develop.latest
- kubectl apply -f my-manifest-using-dummyapp:develop.latest

TODO: Start here tomorrow, look at kubectl manifests

- List images in cluster node with: `docker exec -it kind-control-plane crictl images`

## Other Kind commands
- kind export logs ~/kindlogs
    - defaults to /tmp/[numbers]

# Glossary
- Cluster: Set of nodes
- Pod: A set of running containers on the cluster. Smallest Kubernetes object.
    - StatefulSet: Set of Pods, providing guarantees about the ordering and uniqueness of the Pods. Like a Deployment, but the Pods inside are persistent.
- Kubelet: Agent running on each node in the cluster. Makes sure containers are running in a Pod
