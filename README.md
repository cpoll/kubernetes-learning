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

- `. kind.sh` to create a kind cluster with ingresses
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

- List images in cluster node with: `docker exec -it kind-control-plane crictl images`

## Other Kind commands
- kind export logs ~/kindlogs
    - defaults to /tmp/[numbers]


# Deploying a Manifest
- kubectl apply -f dummy_deploy [-r --recursive]

- kubectl get services
    - Lists all services, including our dummyapp service
- kubectl get deployment dummyapp
- kubectl delete deployment dummyapp
- kubectl delete service dummyapp

- Delete by label:
    kubectl delete deployment,services,persistentvolumeclaim -l app=dummyapp

- Delete by file:
    kubectl delete -f dummy_deploy

- kubectl get all [-l app=dummyapp]

## Troubleshooting failures
https://learnk8s.io/troubleshooting-deployments
- Look at the state of the app
    kubectl get all [-l app=dummyapp]
        - all doesn't show everything, you need to do something cute like:
            kubectl api-resources --verbs=list --namespaced -o name \
  | xargs -n 1 kubectl get --show-kind --ignore-not-found -l app=dummyapp
            see: https://github.com/kubernetes/kubectl/issues/151
- Look for error messages
    kubectl describe pod dummyapp
        - Returns "0/1 nodes are available: 1 persistentvolumeclaim "dummyapp-pv-claim" not found."
        - That's because I spelled it "dummy-app-pv-claim"
- Fix and rerun manifest
    kubectl apply -f dummy_deploy
        - Note that we still have the misspelled persistentvolume, so you'll have to delete that
- Find the next error
    - kubectl describe pod dummyapp gives "Error: secret "dummy-password" not found"
    - kubectl get secrets
    - Add a secret
        kubectl create secret generic dummy-username-password \
            --from-literal=username=dummy-user \
            --from-literal=password='hunter2'

        "Opaque" is the type for "generic" arbitrary user secrets
        TODO: Look into adding secrets with Kustomize
    - You can read the secret if you have access to the cluster
        kubectl get secrets/dummy-username-password -o yaml
            The secrets are hidden in this format, but you can get them with --template. The output is just base64 encoded, you can decode in a one-liner
        kubectl get secrets/dummy-username-password --template={{.data.password}} | base64 -D

    - read a pod config with
        kubectl get pods dummyapp -o yaml
        kubectl edit pods dummyapp

## Deploying changes
    - Kubernetes will pull upon Pod creation only if image is tagged :latest or imagePullPolicy: Always is specified.

    - kubectl apply -f dummy_deploy --record
    - kubectl scale deploy/dummyapp --replicas=3
    - kubectl rollout status deploy/dummyapp
    - kubectl rollout pause deploy/dummyapp
    - kubect rollout resume deploy/dummyapp
    - kubectl rollout history
    - kubectl rollout undo
    - kubectl rollout undo --to-revision=1

# Networking
https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

- port in pod vs port in service?

- TODO: What happens if you deploy two pods that are both on port 80 (with different services)
    - Each pod gets their own IP address?

- LoadBalancer external IP is pending. > Kubernetes does not offer an implementation of network load balancers (Services of type LoadBalancer) for bare-metal clusters. The implementations of network load balancers that Kubernetes does ship with are all glue code that calls out to various IaaS platforms (GCP, AWS, Azure…). If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the “pending” state indefinitely when created. - https://metallb.universe.tf/

# Ingresses
- Ingress objects use ingress controllers, which must be created seperately on your cluster.
- If you go nginx, there are two ingresses, the Kube-maintained one and the Nginx-maintained one. The nginx one is not free.

## Installing nginx ingress on Kind
https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx

- kind.sh

- kubectl apply -f echo_service

- should output "foo"
curl localhost/foo
- should output "bar"
curl localhost/bar



## Picking an Ingress
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- https://medium.com/flant-com/comparing-ingress-controllers-for-kubernetes-9b397483b46b
- https://itnext.io/kubernetes-ingress-controllers-how-to-choose-the-right-one-part-1-41d3554978d2

# Helm
- helm install -f myvalues.yaml mydeployname ./path/to/chartdir
    - TODO: Is it correct to have a myvalues.yaml for each deployment? Can we put the name in the yaml file?
    - helm install -f dummy_chart/values.yaml dummy_helm_deploy ./dummy_chart
- helm uninstall

- helm history RELEASE_NAME
- helm rollback <RELEASE> [REVISION] [flags]
- helm create <chartname>

# Kapitan
- Basic structure created with `kapitan init`
- Render target with `kapitan inventory -t my_target`
- Search inventories where variable is declared with `kapitan searchvar parameters.target_name`
- kapitan compile

## Default Hierarchy
/components
    jsonnet files which each correspond to an application
/inventory
    /classes
        inventory values to be inherited by targets
    /targets
        e.g. dev.yml, staging.yml, prod.yml
/templates
    Jinja2 and Kadet templates
/refs
    secrets referenced in inventory

# Glossary
- Cluster: Set of nodes
- Pod: A set of running containers on the cluster. Smallest Kubernetes object.
    - StatefulSet: Set of Pods, providing guarantees about the ordering and uniqueness of the Pods. Like a Deployment, but the Pods inside are persistent.
- Kubelet: Agent running on each node in the cluster. Makes sure containers are running in a Pod

- Deployment: Definition for creating Pods
- Service: Internal load balancer, routes traffic to Pods
- Ingress: Load balancer at the edge of the cluster

# Other reading
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/
https://learnk8s.io/troubleshooting-deployments


# TODOS:
- Using a develop.latest tag sucks (always haßßs), use a guid if you're not running a persistent CI
- Can/should we get kind to pull from a Docker repo (including the local repo)?
- https://kind.sigs.k8s.io/docs/user/local-registry/
