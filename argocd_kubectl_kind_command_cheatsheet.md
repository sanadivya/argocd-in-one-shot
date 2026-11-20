# Argo CD + kubectl + kind Command Cheat Sheet

A practical command reference for day-to-day Kubernetes and GitOps
practice.

**Environment note:** Examples use Windows Command Prompt
(`C:\Users\...>`), which matches the current practice setup. Most
commands also work in PowerShell. Run commands from a terminal where
`kubectl`, `kind`, `argocd`, and (where needed) Docker are installed.

> Replace values such as `<cluster-name>`, `<namespace>`, `<app-name>`,
> `<context>`, and `<file.yaml>` with your actual names. Do not type the
> angle brackets.

------------------------------------------------------------------------

# 1. kind --- Local Kubernetes clusters

## Check installation and version

``` cmd
REM Check the installed kind version.
kind version
REM Confirm Docker is installed and its engine is reachable.
docker version
REM Check the installed kubectl client version.
kubectl version --client
```

## Create a cluster

``` cmd
REM Create a local Kubernetes cluster; options select its name, node image, or config file.
kind create cluster
REM Create a local Kubernetes cluster; options select its name, node image, or config file.
kind create cluster --name k8s-demo
REM Create a local Kubernetes cluster; options select its name, node image, or config file.
kind create cluster --name k8s-demo --image kindest/node:v1.37.0
REM Create a local Kubernetes cluster; options select its name, node image, or config file.
kind create cluster --name k8s-demo --config kind-config.yaml
```

Example `kind-config.yaml` with one control-plane and two workers:

``` yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

## List, inspect, and get cluster details

``` cmd
REM List local kind clusters.
kind get clusters
REM List the Docker containers used as nodes by kind clusters.
kind get nodes
REM Print the kubeconfig connection details for the named kind cluster.
kind get kubeconfig --name k8s-demo
REM List running Docker containers, including kind node containers.
docker ps
REM Display the control-plane and cluster service addresses.
kubectl cluster-info
REM List cluster nodes and optionally show their addresses and labels.
kubectl get nodes -o wide
```

## Load a locally built image into kind

``` cmd
REM Build a local container image with the specified name and tag.
docker build -t myapp:v1 .
REM Load a locally built Docker image into the kind cluster without pushing it to a registry.
kind load docker-image myapp:v1 --name k8s-demo
```

For a Deployment using a local kind-loaded image, commonly set:

``` yaml
image: myapp:v1
imagePullPolicy: IfNotPresent
```

If Kubernetes tries to pull a local-only image, check the image name/tag
and `imagePullPolicy`.

## Export and merge kubeconfig

``` cmd
REM Write or merge the kind cluster credentials into a kubeconfig file.
kind export kubeconfig --name k8s-demo
REM Write or merge the kind cluster credentials into a kubeconfig file.
kind export kubeconfig --name k8s-demo --kubeconfig C:\Users\<you>\.kube\config
```

## Delete a cluster

``` cmd
REM Delete the local kind cluster and its cluster data.
kind delete cluster
REM Delete the local kind cluster and its cluster data.
kind delete cluster --name k8s-demo
```

Deleting a kind cluster deletes its cluster resources and local cluster
data. It does not delete remote clusters.

------------------------------------------------------------------------

# 2. kubectl --- Contexts and cluster access

## Verify which cluster you are connected to

``` cmd
REM Show the active Kubernetes context so you know which cluster commands will affect.
kubectl config current-context
REM List configured contexts so you can choose the correct cluster.
kubectl config get-contexts
REM Inspect the kubeconfig settings and cluster credentials.
kubectl config view
REM Show configuration for only the active context.
kubectl config view --minify
REM Display the control-plane and cluster service addresses.
kubectl cluster-info
REM List cluster nodes and optionally show their addresses and labels.
kubectl get nodes -o wide
```

**Important:** Before applying or deleting resources, check
`kubectl config current-context`. This prevents accidentally changing
the wrong cluster.

## Switch contexts

``` cmd
REM Switch kubectl to the specified cluster context.
kubectl config use-context kind-k8s-demo
REM Switch kubectl to the specified cluster context.
kubectl config use-context <context-name>
REM Give a kubeconfig context a clearer name.
kubectl config rename-context <old-name> <new-name>
REM Remove a context entry from kubeconfig.
kubectl config delete-context <context-name>
```

## Namespaces

``` cmd
REM List namespaces in the cluster.
kubectl get namespaces
REM List namespaces in the cluster.
kubectl get ns
REM Create a namespace to isolate a group of Kubernetes resources.
kubectl create namespace dev
REM Inspect namespace details and related events.
kubectl describe namespace dev
REM Delete the namespace and all namespaced resources inside it.
kubectl delete namespace dev
REM Set the default namespace for commands using the current context.
kubectl config set-context --current --namespace=dev
REM Show configuration for only the active context.
kubectl config view --minify
```

The namespace set on the current context is only a default for commands
that omit `-n`; it does not move resources between namespaces.

------------------------------------------------------------------------

# 3. kubectl --- Everyday resource commands

## Get and list resources

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get all
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get all -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -o wide -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get deployments -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get services -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get svc -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get ingress -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get configmaps -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get secrets -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pvc -A
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pv
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get storageclass
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

`kubectl get all` is not literally every Kubernetes resource type. Query
a specific resource when you need a complete view.

## Describe and inspect YAML

``` cmd
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe pod <pod-name> -n <namespace>
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe deployment <deployment-name> -n <namespace>
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe service <service-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pod <pod-name> -n <namespace> -o yaml
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get deployment <deployment-name> -n <namespace> -o yaml
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pod <pod-name> -n <namespace> -o json
```

## Apply, update, and delete manifests

``` cmd
REM Create or update resources from the supplied manifest file or directory.
kubectl apply -f deployment.yaml
REM Create or update resources from the supplied manifest file or directory.
kubectl apply -f folder\
REM Create or update resources from the supplied manifest file or directory.
kubectl apply -f deployment.yaml -n <namespace>
REM Preview the changes that applying the manifest would make.
kubectl diff -f deployment.yaml
REM Delete resources declared in the supplied manifest file or directory.
kubectl delete -f deployment.yaml
REM Delete resources declared in the supplied manifest file or directory.
kubectl delete -f folder\
REM Delete the named Kubernetes resource.
kubectl delete pod <pod-name> -n <namespace>
REM Delete the named Kubernetes resource.
kubectl delete deployment <deployment-name> -n <namespace>
REM Delete the named Kubernetes resource.
kubectl delete service <service-name> -n <namespace>
```

`apply` creates or updates resources declared in the file. `delete -f`
deletes the resources described by that file.

## Create resources imperatively

``` cmd
REM Create a Deployment that manages Pods for the specified container image.
kubectl create deployment nginx --image=nginx
REM Create a Deployment that manages Pods for the specified container image.
kubectl create deployment nginx --image=nginx -n dev
REM Create a Service that provides network access to the selected workload.
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
REM Create a ConfigMap from a file or literal key-value setting.
kubectl create configmap app-config --from-literal=MODE=dev
REM Create a Secret for sensitive configuration; avoid exposing real values in shell history.
kubectl create secret generic app-secret --from-literal=PASSWORD=<value>
REM Create a namespace to isolate a group of Kubernetes resources.
kubectl create namespace test
```

Avoid putting real passwords directly in shell history or shared command
logs. Use an approved secret-management workflow for real environments.

------------------------------------------------------------------------

# 4. kubectl --- Deployments, Pods, and scaling

## Scale and rollout

``` cmd
REM Change the desired replica count for the Deployment.
kubectl scale deployment <deployment-name> --replicas=3 -n <namespace>
REM Wait for the Deployment rollout to complete and report its status.
kubectl rollout status deployment/<deployment-name> -n <namespace>
REM View rollout revisions to understand previous Deployment updates.
kubectl rollout history deployment/<deployment-name> -n <namespace>
REM View rollout revisions to understand previous Deployment updates.
kubectl rollout history deployment/<deployment-name> --revision=2 -n <namespace>
REM Roll the Deployment back to its previous revision.
kubectl rollout undo deployment/<deployment-name> -n <namespace>
REM Trigger a rolling restart of the Deployment's Pods.
kubectl rollout restart deployment/<deployment-name> -n <namespace>
REM Update a container image in the Deployment and trigger a rollout.
kubectl set image deployment/<deployment-name> <container-name>=<image>:<tag> -n <namespace>
```

## Pause and resume a rollout

``` cmd
REM Pause a Deployment rollout while you inspect or coordinate changes.
kubectl rollout pause deployment/<deployment-name> -n <namespace>
REM Resume a previously paused Deployment rollout.
kubectl rollout resume deployment/<deployment-name> -n <namespace>
```

## Pods and containers

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -n <namespace> -o wide
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe pod <pod-name> -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs <pod-name> -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs <pod-name> -c <container-name> -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs <pod-name> --previous -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs -f <pod-name> -n <namespace>
REM Run a command inside a running container, often to open a troubleshooting shell.
kubectl exec -it <pod-name> -n <namespace> -- sh
REM Run a command inside a running container, often to open a troubleshooting shell.
kubectl exec -it <pod-name> -n <namespace> -- bash
REM Delete the named Kubernetes resource.
kubectl delete pod <pod-name> -n <namespace>
```

Use `--previous` to inspect logs from the previous crashed container
instance. A deleted Pod managed by a Deployment/StatefulSet may be
recreated automatically.

------------------------------------------------------------------------

# 5. kubectl --- Networking and troubleshooting

## Services and endpoints

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get svc -A
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe svc <service-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get endpoints <service-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get endpointslices -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get ingress -A
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe ingress <ingress-name> -n <namespace>
```

## Port forwarding

``` cmd
REM Forward a local port to a Pod, Service, or Deployment for temporary access.
kubectl port-forward deployment/<deployment-name> 8080:80 -n <namespace>
REM Forward a local port to a Pod, Service, or Deployment for temporary access.
kubectl port-forward service/<service-name> 8080:80 -n <namespace>
REM Forward a local port to a Pod, Service, or Deployment for temporary access.
kubectl port-forward pod/<pod-name> 8080:80 -n <namespace>
```

Open `http://localhost:8080` while the port-forward command is running.
Stop it with `Ctrl+C`.

## Debugging a failing Pod

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -n <namespace>
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe pod <pod-name> -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs <pod-name> -n <namespace>
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs <pod-name> --previous -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pod <pod-name> -n <namespace> -o yaml
```

Common status clues:

-   `ImagePullBackOff` / `ErrImagePull`: image name, tag, registry
    permissions, or pull secret.
-   `CrashLoopBackOff`: container starts and exits; inspect current and
    previous logs.
-   `Pending`: scheduling, resource requests, PVC, node selector,
    affinity, or taints.
-   `CreateContainerConfigError`: missing ConfigMap/Secret/key or
    invalid container configuration.
-   `Running` does not always mean the application is healthy; check
    readiness and liveness probes.

## Resource usage (requires Metrics Server)

``` cmd
REM View CPU and memory usage; this requires Metrics Server.
kubectl top nodes
REM View CPU and memory usage; this requires Metrics Server.
kubectl top pods -A
REM View CPU and memory usage; this requires Metrics Server.
kubectl top pods -n <namespace>
```

------------------------------------------------------------------------

# 6. kubectl --- ConfigMaps, Secrets, and storage

## ConfigMaps

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get configmaps -A
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe configmap <configmap-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get configmap <configmap-name> -n <namespace> -o yaml
REM Create a ConfigMap from a file or literal key-value setting.
kubectl create configmap app-config --from-file=app.properties
REM Create a ConfigMap from a file or literal key-value setting.
kubectl create configmap app-config --from-literal=LOG_LEVEL=debug
REM Delete the named Kubernetes resource.
kubectl delete configmap <configmap-name> -n <namespace>
```

## Secrets

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get secrets -n <namespace>
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe secret <secret-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get secret <secret-name> -n <namespace> -o yaml
REM Create a Secret for sensitive configuration; avoid exposing real values in shell history.
kubectl create secret generic app-secret --from-literal=API_KEY=<value>
```

Secret values in YAML are base64-encoded, not encrypted merely because
they are base64. Protect access to Secrets and enable appropriate
encryption at rest in real clusters.

## Persistent storage

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pv
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pvc -A
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe pvc <pvc-name> -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get storageclass
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe pv <pv-name>
```

A PVC in `Pending` may be waiting for a matching PV, a StorageClass
provisioner, or a consumer Pod when the StorageClass uses
`WaitForFirstConsumer`.

------------------------------------------------------------------------

# 7. kubectl --- Labels, annotations, and scheduling

``` cmd
REM List cluster nodes and optionally show their addresses and labels.
kubectl get nodes --show-labels
REM Add, change, or remove a label used for selection and scheduling.
kubectl label node <node-name> role=worker
REM Add, change, or remove a label used for selection and scheduling.
kubectl label node <node-name> role-
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods --show-labels -n <namespace>
REM Add, change, or remove a label used for selection and scheduling.
kubectl label pod <pod-name> app=demo -n <namespace>
REM Attach non-identifying metadata to a resource.
kubectl annotate deployment <deployment-name> owner=team-a -n <namespace>
REM Mark a node unschedulable so new Pods are not placed there.
kubectl cordon <node-name>
REM Allow new Pods to be scheduled on the node again.
kubectl uncordon <node-name>
REM Safely evict eligible Pods before node maintenance; check disruption impact first.
kubectl drain <node-name> --ignore-daemonsets
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe node <node-name>
```

`drain` evicts eligible workloads and can disrupt applications. Review
PodDisruptionBudgets and workload replicas before using it. On a local
kind cluster, use it only as a deliberate scheduling exercise.

------------------------------------------------------------------------

# 8. Argo CD --- CLI login and basic access

The Argo CD CLI is commonly named `argocd`.

``` cmd
REM Check the Argo CD CLI and/or server version.
argocd version
REM Check the Argo CD CLI and/or server version.
argocd version --client
REM Authenticate the CLI to the Argo CD API server.
argocd login <ARGOCD_SERVER>
REM Authenticate the CLI to the Argo CD API server.
argocd login localhost:8080 --username admin --password <password> --insecure
REM Inspect the currently authenticated Argo CD account.
argocd account get-user-info
REM Inspect the currently authenticated Argo CD account.
argocd account list
REM List Kubernetes clusters registered with Argo CD.
argocd cluster list
REM List Git or Helm repositories configured in Argo CD.
argocd repo list
```

For a local Argo CD installation, port-forward the API server in one
terminal:

``` cmd
REM Forward a local port to a Pod, Service, or Deployment for temporary access.
kubectl port-forward svc/argocd-server 8080:443 -n argocd
```

Then log in from another terminal:

``` cmd
REM Authenticate the CLI to the Argo CD API server.
argocd login localhost:8080 --username admin --password <password> --insecure
```

The `--insecure` option skips certificate verification. It is often used
for local self-signed lab setups, not as a general production
recommendation.

## Get the initial admin password (if admin is enabled)

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

The output is base64-encoded. On Windows PowerShell, decode it with:

``` powershell
REM Decode the initial Argo CD password from base64 in PowerShell.
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("<base64-output>"))
```

If the initial secret is absent, the password may already have been
changed, admin may be disabled, or the installation method may differ.

------------------------------------------------------------------------

# 9. Argo CD --- Applications

## List and inspect applications

``` cmd
REM List Argo CD Applications and their sync and health states.
argocd app list
REM Inspect an Application's source, destination, sync, health, and operation details.
argocd app get <app-name>
REM Inspect an Application's source, destination, sync, health, and operation details.
argocd app get <app-name> --show-operation
REM Compare the desired Git manifests with the live cluster resources.
argocd app diff <app-name>
REM Display the rendered Kubernetes manifests Argo CD expects to apply.
argocd app manifests <app-name>
REM Show deployment history and revisions for an Application.
argocd app history <app-name>
REM List the Kubernetes resources managed by an Application.
argocd app resources <app-name>
```

## Create an application

``` cmd
REM Create an Argo CD Application that connects a repository path to a destination cluster and namespace.
argocd app create <app-name> --repo <git-repo-url> --path <folder-path> --dest-server https://kubernetes.default.svc --dest-namespace <namespace>
```

For a Git branch:

``` cmd
REM Create an Argo CD Application that connects a repository path to a destination cluster and namespace.
argocd app create <app-name> --repo <git-repo-url> --path <folder-path> --revision main --dest-server https://kubernetes.default.svc --dest-namespace <namespace>
```

## Sync and refresh

``` cmd
REM Apply the desired Git state; --prune can also delete resources removed from Git.
argocd app sync <app-name>
REM Apply the desired Git state; --prune can also delete resources removed from Git.
argocd app sync <app-name> --prune
REM Ask Argo CD to refresh its comparison of Git and live cluster state.
argocd app refresh <app-name>
REM Ask Argo CD to refresh its comparison of Git and live cluster state.
argocd app refresh <app-name> --hard
REM Wait until the Application reaches the requested sync or health condition.
argocd app wait <app-name>
REM Wait until the Application reaches the requested sync or health condition.
argocd app wait <app-name> --health
```

-   `refresh` asks Argo CD to compare Git and cluster state.
-   `--hard` also refreshes the cached repository/manifest data.
-   `sync` applies the desired manifests.
-   `--prune` allows sync to delete resources that are no longer present
    in the desired source. Use carefully.

## Automatic sync settings

``` cmd
REM Change Application settings such as automated sync, pruning, or self-healing.
argocd app set <app-name> --sync-policy automated
REM Change Application settings such as automated sync, pruning, or self-healing.
argocd app set <app-name> --sync-policy none
REM Change Application settings such as automated sync, pruning, or self-healing.
argocd app set <app-name> --auto-prune
REM Change Application settings such as automated sync, pruning, or self-healing.
argocd app set <app-name> --self-heal
REM Change Application settings such as automated sync, pruning, or self-healing.
argocd app set <app-name> --allow-empty
```

`--auto-prune`, `--self-heal`, and `--allow-empty` are options used with
automated sync configuration. Check the installed Argo CD CLI version if
an option differs.

## Delete an application

``` cmd
REM Delete the Argo CD Application; cascading deletion may also remove its managed resources.
argocd app delete <app-name>
REM Delete the Argo CD Application; cascading deletion may also remove its managed resources.
argocd app delete <app-name> --cascade
REM Delete the Argo CD Application; cascading deletion may also remove its managed resources.
argocd app delete <app-name> --cascade=false
```

A cascading deletion can delete the application's managed Kubernetes
resources. Non-cascading deletion removes the Argo CD Application object
but leaves managed resources behind. Also check the Application's
finalizers and parent Application/ApplicationSet before deleting.

------------------------------------------------------------------------

# 10. Argo CD --- Repositories and clusters

## Repositories

``` cmd
REM List Git or Helm repositories configured in Argo CD.
argocd repo list
REM Inspect a configured repository connection.
argocd repo get <repo-url>
REM Register a Git repository so Argo CD can read its manifests.
argocd repo add <repo-url> --type git
REM Remove a repository registration from Argo CD.
argocd repo rm <repo-url>
```

For a private repository, use the authentication method approved for
your environment. Avoid placing long-lived credentials in shell history.

## Register and inspect clusters

``` cmd
REM List Kubernetes clusters registered with Argo CD.
argocd cluster list
REM Register a kubeconfig context as a destination cluster for Argo CD.
argocd cluster add <kubectl-context-name>
REM Inspect a registered destination cluster.
argocd cluster get <cluster-server>
REM Remove a destination cluster registration from Argo CD.
argocd cluster rm <cluster-server>
```

`argocd cluster add` normally creates or updates an Argo CD access
ServiceAccount and RBAC resources in the target cluster. Review the
permissions it grants; do not automatically accept broad cluster-admin
access for production.

------------------------------------------------------------------------

# 11. Argo CD --- ApplicationSet and App of Apps

## ApplicationSets

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get applicationsets -n argocd
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe applicationset <applicationset-name> -n argocd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get applicationset <applicationset-name> -n argocd -o yaml
REM Create or update resources from the supplied manifest file or directory.
kubectl apply -f applicationset.yaml
REM Delete resources declared in the supplied manifest file or directory.
kubectl delete -f applicationset.yaml
```

## Applications

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get applications -n argocd
REM Show detailed resource configuration, status, conditions, and events.
kubectl describe application <app-name> -n argocd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get application <app-name> -n argocd -o yaml
REM Create or update resources from the supplied manifest file or directory.
kubectl apply -f application.yaml
REM Delete the named Kubernetes resource.
kubectl delete application <app-name> -n argocd
```

If an Application is recreated after deletion, check whether an
ApplicationSet or parent "app-of-apps" Application is generating it.
Remove or change the parent source definition if you want it to stay
deleted.

## Argo CD controller and server logs

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -n argocd
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs deployment/argocd-server -n argocd
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs deployment/argocd-repo-server -n argocd
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs deployment/argocd-applicationset-controller -n argocd
REM Read container output; use -f to follow logs and --previous for the last crashed instance.
kubectl logs statefulset/argocd-application-controller -n argocd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get events -n argocd --sort-by=.metadata.creationTimestamp
```

------------------------------------------------------------------------

# 12. A useful daily GitOps workflow

1.  Confirm the Kubernetes context:

``` cmd
REM Show the active Kubernetes context so you know which cluster commands will affect.
kubectl config current-context
REM List cluster nodes and optionally show their addresses and labels.
kubectl get nodes
```

2.  Check Argo CD applications:

``` cmd
REM List Argo CD Applications and their sync and health states.
argocd app list
REM Inspect an Application's source, destination, sync, health, and operation details.
argocd app get <app-name>
```

3.  Make the change in the Git repository and commit/push it:

``` cmd
REM Review changed and untracked files before committing.
git status
REM Stage the selected changes for the next commit.
git add .
REM Create a commit containing the staged changes.
git commit -m "update application configuration"
REM Push local commits to the configured remote repository.
git push
```

4.  Ask Argo CD to refresh if you need to check immediately:

``` cmd
REM Ask Argo CD to refresh its comparison of Git and live cluster state.
argocd app refresh <app-name>
REM Compare the desired Git manifests with the live cluster resources.
argocd app diff <app-name>
```

5.  If auto-sync is disabled, sync manually:

``` cmd
REM Apply the desired Git state; --prune can also delete resources removed from Git.
argocd app sync <app-name>
```

6.  Verify the resulting Kubernetes resources:

``` cmd
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get all -n <namespace>
REM List the requested Kubernetes resource type; add -n or -A to control namespace scope.
kubectl get pods -n <namespace> -o wide
REM Wait for the Deployment rollout to complete and report its status.
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

**GitOps principle:** For resources managed by Argo CD, make the lasting
change in Git. A direct `kubectl edit` or `kubectl apply` may be
reverted by Argo CD when self-heal is enabled, or may leave the
application OutOfSync.

------------------------------------------------------------------------

# 13. Quick troubleshooting checklist

  ------------------------------------------------------------------------------------------
  Symptom                             First commands to run
  ----------------------------------- ------------------------------------------------------
  Wrong cluster                       `kubectl config current-context`

  Pod not starting                    `kubectl describe pod <pod> -n <ns>`

  Container keeps restarting          `kubectl logs <pod> --previous -n <ns>`

  Image pull failure                  `kubectl describe pod <pod> -n <ns>`

  Service not reachable               `kubectl get svc,endpoints -n <ns>`

  PVC pending                         `kubectl describe pvc <pvc> -n <ns>`

  Argo app OutOfSync                  `argocd app diff <app>`

  Argo app not updating after push    `argocd app refresh <app> --hard`

  App keeps reappearing               `kubectl get applications,applicationsets -n argocd`

  Need Argo CD component logs         `kubectl get pods -n argocd` then
                                      `kubectl logs ... -n argocd`
  ------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 14. Command safety reminders

-   Check `kubectl config current-context` before applying, patching, or
    deleting.
-   Prefer `kubectl diff -f <file>` before `kubectl apply -f <file>`.
-   `kubectl delete namespace`, `kind delete cluster`, and cascading
    Argo CD app deletion can remove many resources.
-   `argocd app sync --prune` can delete resources absent from Git.
-   Do not commit Kubernetes Secret values, kubeconfig files, access
    keys, or passwords to Git.
-   In GitOps, Git is the source of truth for the resources managed by
    Argo CD.
-   Command flags can vary by Kubernetes/Argo CD version. If a flag is
    rejected, check `kubectl <command> --help` or
    `argocd <command> --help`.
