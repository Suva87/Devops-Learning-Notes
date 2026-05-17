# Introduction:

## What is CI/CD?

CI: Continious Intergration
CD: Continuous Deployment

## What problem does CI/CD solve?

Before CI/CD, many teams used to work like this:
developer writes code
developer builds app manually
developer runs tests manually
developer creates Docker image manually
developer pushes image manually
developer runs kubectl apply -f ... manually

But after some time, problems start:
people forget steps
people deploy wrong version
two developers overwrite each other
cluster state becomes different from Git state
production becomes messy
rollback becomes difficult
So CI/CD exists to remove manual repeated work and make the flow predictable.

## What is CI?

CI means: when code changes, the system should automatically do the integration work.
That usually means things like:
pull the latest code
install dependencies
run tests
build the app
build Docker image
push Docker image to registry
So CI is mostly about: “Is the new code valid and buildable?”
CI = code checking + build preparation

## What is CD?

Once code is already built and ready, now it must reach the environment.
That means:
update deployment config
apply manifests
sync Helm chart
deploy new image to Kubernetes
verify target state

So CD is mostly about: “Take the approved version and move it into the environment safely.”
CD = delivery/deployment of the built application

# GITOPS:

Old way: You do: kubectl apply -f deployment.yaml . That means you are directly changing the cluster.
In GitOps = “Git decides what the cluster should look like.”
GitOps way: You do:
change YAML / Helm values in Git
commit
push

ArgoCD notices the change
ArgoCD syncs the cluster

That means Git becomes the control point.

It works on DVAO principle:
Declarative - Version Controlled - Automated - Observable

# ARGOCD INTRO:

ArgoCD is mainly a CD tool for Kubernetes. It follows the GitOps model.

ArgoCD uses Git as the source of truth for what should be running in the cluster. The docs describe Argo CD as following the GitOps pattern and supporting manifests from Helm charts, Kustomize, plain YAML, and more.

It is mainly for:
watching Git
comparing Git state with cluster state
syncing the cluster to match Git
ArgoCD = Kubernetes CD tool based on GitOps

it solves a very common Kubernetes problem: “What is actually supposed to be running right now?”

## Desired State vs Live State:

Desired state: The state defined in Git (what should be running)
live state: The actual state in Kubernetes

ArgoCD compares Desired vs Live state

## Drift:

When cluster state differs from Git state

## Remember: ArgoCD does NOT deploy apps directly. It continuously checks: Git (desired state) vs Cluster (live state). If different → it syncs them.

## Sync Modes:

Manual Sync: User clicks "Sync"

Auto Sync: ArgoCD automatically applies changes

Options:

- prune → delete old resources
- selfHeal → fix drift

## ArgoCD basic working flow:

.) Step 1:
You have a Git repository. Inside it, you keep things like:
Helm chart
Kubernetes YAML
values files
app configuration
.) Step 2:
ArgoCD is installed in Kubernetes.
.) Step 3:
You create an ArgoCD Application.
This tells ArgoCD:
which Git repo to watch
which path to read
which branch/tag/revision to use
which cluster to deploy to
which namespace to deploy into
.) Step 4:
ArgoCD reads that source.
.) Step 5:
ArgoCD compares:
desired state from Git
actual live state in cluster
If they are different, ArgoCD marks the app as OutOfSync.
Then either:
you sync manually
or auto-sync is enabled and ArgoCD syncs automatically

# BASIC INFRASTRUCTURE OF ARGOCD:

        ARGO CD has three main things:
            1) API Server -  - Handles UI, CLI, API requests - Entry point (UI + CLI + API) - Handles user actions (sync, login, app status)
            2) Repository Server - Clones Git repo, it reads the manifest files, Renders Helm charts, Generates final YAML
            3) Application Controller - Compares Git state vs cluster state, - Detects drift

# SET UP:

For now we will use kind cluster to learn.

Step 1:
Create a EC2 instance: min T2 large

Step 2: refer to previous notes for these pre-requisite installations
-> Update the system: sudo apt-get update
=> Make sure docker is installed:
=> make sure kind is installed:
=> make sure kubectl is installed

Step 3: make a kind cluster: \*\*\*\*... now this config is a little different.. with some additional key values... kind-config.yaml

```YAML
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    networking:      ### this is new
        apiServerAddress: "172.31.95.138" ### this is your EC2 private address
        apiServerPort: 33893
    nodes:
        - role: control-plane
        image: kindest/node:v1.33.1
        - role: worker
        image: kindest/node:v1.33.1
        - role: worker
        image: kindest/node:v1.33.1
```

now:

```
kind create cluster --name argocd-cluster --config=kind-config.yaml

```

Step 4: Installation through Helm repo:
=> install helm: refer to k8s notes for installation guide
=> Add Argo helm repo:
->

```
helm repo add argo https://argoproj.github.io/argo-helm
```

->

```
helm repo update
```

=> create a ns:

```
kubectl create ns argocd
```

=> install argocd:

```
helm install argocd argo/argo-cd -n argocd
```

=> verify installation

```
kubectl get pods -n argocd
kubectl get svc -n argocd
```

Now, if you want to access the UI of argocd:
=>

```
kubectl port-forward service/argocd-server -n argocd 8080:443 --address=0.0.0.
```

=> then edit inbound rules on your ec2 and add 8080 and you can then access argocd
ui with your public ip of ec2 on port 8080.
=> to get initial password;

```
kubectl get secret argocd-initial-admin-secret -n argocd \ -o jsonpath="{.data.password}" | base64 -d && echo
```

=> after login go to "userinfo" -->> "update password"

There can be another method og login through kubectl as well..

# METHODS TO USE ARGO:

.) Through UI
.) Through CLI
.) Through manifest files - Declarative approach

# UNDERSTANDING THE UI COMPONENTS:

First go to settings in the UI and understand each of the things..

## PROJECTS:

    this creates a group, just like namespace in K8S..
        Project = Security + Policy Boundary
        👉 It is access control + restriction layer

    it says:
        - which repos can be used
        - which clusters/namespaces can be deployed to
        - what resources are allowed

    Think like this:
        👉 Application = “WHAT to deploy”
        👉 Project = “WHERE and HOW you are allowed to deploy”

    🎯 Why Projects exist (VERY IMPORTANT) ?

        Without Projects:
            Any app can deploy anywhere
            Any repo can be used
            Any namespace can be targeted
        Now imagine: ❌ Developer accidentally deploys testing code to production
        ✅ With Projects:
            You define:
                Project: dev
                - can deploy only to namespace: dev
                - can use only dev repo

                Project: prod
                - can deploy only to namespace: prod
                - only approved repos allowed

    🧱  What does a Project control?

        🔹 (1) Source Repositories
                    sourceRepos: - https://github.com/company/*
                👉 Which Git repos are allowed
        🔹 (2) Destination (cluster + namespace)
                    destinations:
                    - namespace: dev
                    server: https://kubernetes.default.svc
                👉 Where deployment is allowed
        🔹 (3) Resource Restrictions
                👉 You can restrict:
                    => allowed resources (Deployment, Service)
                    => blocked resources (ClusterRole, CRD)
        🔹 (4) RBAC (Access Control)
                👉 Who can:
                   => view apps
                   => sync apps
                   => modify apps

        Project → controls rules
        Application → follows those rules

    📌 Default Project (important observation):

        When you install ArgoCD, you will see: default project. 👉 This allows everything.
        That means:
            any repo - any namespace - any resource
        👉 Default project is NOT secure

# FIRST APP IN DEFAULT PROJECT:

👉 Step 1:

    👉 Create a project :

```YAML

apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
    name: dev-project
    namespace: argocd

spec:
    sourceRepos:
        - https://github.com/your-org/*

    destinations:
        - namespace: dev
            server: https://kubernetes.default.svc

    clusterResourceWhitelist:
        - group: '*'
            kind: '*'
```

But for the below example, as a beginner we have created everything in default project for now..

=> Login to argocd on the ec2:

`      👉 argocd login <ec2-public-ip>:8080 --username admin --password <your-password> --insecure`

as of now we are not using https, so --insecure..

you can get the user info by: argocd get user info

=> to get initial password:

```
kubectl get secret argocd-initial-admin-secret -n argocd \ -o jsonpath="{.data.password}" | base64 -d && echo`
```

=> later on change the password through the UI

=> Add your kind cluster with argocd:
-> find out your cluster name:

```
kubectl config get-contexts
```

-> add the cluster:

```
argocd cluster add <config-cluster-name> --name <cluster-name-that -you-choose> --insecure
```

eg:

```
argocd cluster add kind-argocd-cluster --name argocd-cluster --insecure
```

this will create a service account as argocd-manager and get the clusterrolebinding done with the service-account..

-> to check:

```
argocd cluster list`
```

=> Now we need to create a Custom Resource (CR) based on an already existing CRD (custom rescource definition) manifest file for the app...

👉 vim online_shop_app.yaml

```YAML
apiVersion: argoproj.io/v1alpha1 #API group for argocd resource
kind: Application #Resource type
metadata:
name: online-shop-app
namespace: argocd
spec:
project: default
source:
    repoURL: https://github.com/<your-username>/<link-to-project>.git #git repo
    targetRevision: main   #git brabch
    path: declarative_approach/online-shop   #path inside your git repo to you deployment
destination:
    server: <Kubernetes_API_server_endpoint>  # get this by : argocd cluster list
    namespace: default    # <argo_cluster_namespace>
syncPolicy:
    automated:
        prune: true
        selfHeal: true

```

```
kubectl apply -f online_shop_app.yaml`
```

1.  Kubernetes receives Application resource
2.  ArgoCD controller watches it
3.  ArgoCD fetches Git repo
4.  ArgoCD generates manifests 5. ArgoCD compares with cluster 6. If OutOfSync → sync (auto/manual)

=> check the port by running:
kubectl get svc ... lets say port 3000

=> port forward:

```
kubectl port-forward service/online-shop-service -n argocd 8083:3000 --address=0.0.0.0 &
```

🔥 VERY IMPORTANT CONCEPTs:
👉 ArgoCD does NOT deploy your code
👉 It deploys what is inside this folder
That folder must contain:
YAML files OR
Helm chart OR
Kustomize
👉 destination: server: <cluster-api> , namespace: default
🧠 Meaning: Deploy into this cluster, Deploy into this namespace
🔥 Important understanding:
=> ArgoCD can deploy to:
same cluster as well as an external cluster
👉 Application YAML = Bridge between Git and Kubernetes

# ARGOCD CORE FEATURES:

## 1) Projects: project.yaml

```YAML
            kind: AppProject
            apiVersion: argoproj.io/v1alpha1
            metadata:
                namespace: argocd
                name: frontend-team
            spec:
                description: This is a sample project
                sourceRepos:
                    - <link-to-your-github>
                destinations:
                    - namespace: frontend
                      server: <Kubernetes_API_server_endpoint>
                clusterResourceWhitelist: ##which clusterwide resources are allowed
                    - group: "*"    ##means all
                      kind: "*"
                namespaceResourceWhitelist:  ##which ns-level resources are allowed
                    - group: "*"
                      kind: "*"

                roles: ##define roles for RBAC within this proj ... will study later
                - name: frontend-admins  #role-name
                  description: Admins for frontend
                  policies: ##policies associated with this role
                    ##syntax: -p, proj:<project-name>:<role-name>, applications, <action>, <project>/<app-name>, <permission(allow|deny)>
                    -p, proj:frontend-team:frontend-admins, applications, *, frontend-team/*, allow     ##means full access
```

```
kubectl apply -f project.yaml -n argocd
```

now go to the UI, and you should see the project added in the project list

```
argocd proj list
```

if you want to check the list of projects through cli

=> now lets say if you want to create a nginx app inside this project:

vim ngnix-app.yaml

```YAML

kind: Application
apiVersion: argoproj.io/v1alpha1
metadata:
    namespace: argocd
    name: nginx-frontend
spec:
    project: frontend-team  ##this is where you define the project
    source:
        repoURL: https://github.com/<your-username>/<link-to-project>.git
        targetRevision: main
        path: ui_approach/ngnix
    destination:
        server: <Kubernetes_API_server_endpoint>
        namespace: frontend
    syncPolicy:
        automated:
            selfHeal: true
            prune: true
        syncOptions:
            - CreateNamespace=true  ###autocreate ns if it doesn't exist.
```

```
kubectl apply -f nginx-app.yaml
```

=> check the service for port no.

```
kubectl get svc -n frontend
```

```
kubectl port-forward svc/<svc-name> -n frontend 8084:<svc-port> --address=0.0.0.0 &
```

## 2) App of Apps:

App of Apps = one parent ArgoCD Application that manages multiple child ArgoCD Applications
normal Application → deploys Kubernetes manifests / Helm / Kustomize
App of Apps → deploys other ArgoCD Applications

For example, your future EMS platform may need:
frontend
backend
mongodb
redis
ingress
monitoring
logging
cert-manager
Instead of creating all those apps manually one by one, you create: one parent app called ems-platform, and that parent manages all child apps.

NOTE: App of Apps is not a special Kubernetes kind. There is no kind called: kind: AppOfApps
It is still just a normal: kind: Application
The difference is in what the source folder contains.

Suppose your Git repo looks like this:

```
argocd/
    apps/
        frontend-app.yaml
        backend-app.yaml
        mongodb-app.yaml
    frontend/
        deployment.yaml
        service.yaml
    backend/
        deployment.yaml
        service.yaml
    mongodb/
        statefulset.yaml
        service.yaml
```

=> frontend/, backend/, mongodb/ contain the real workloads
=> apps/ contains ArgoCD child Application YAMLs
=> parent app points to apps/

Example with Explanation:

Suppose we have 3 apps..

1. Apache app -- That has its own deployment.yaml and services.yaml etc..
   We create its apache-app.yaml in the normal way.. define the name as apache-app-child in metadata
2. Ngnix App -- That has its own deployment.yaml and services.yaml etc..
   We create its nginx-app.yaml in the normal way.. define the name as nginx-app-child in metadata
3. Online-shop-app -- That has its own deployment.yaml and services.yaml etc..
   We create its online-shop-app.yaml in the normal way.. define the name as online-shop-app-child in metadata
4. Now no need to apply them diffrently, so we just create a parent app: root_app.yaml
   Here, in the name we use the name root, like name: root-app,
   and
   the source->
   repoURL: <Repo-Containing-Child-Apps>
   path: <link-to-the-folder-which-contains-all-the-child-app>

So now, you just need to apply: kubectl apply -f root-app.yaml -n argocd
All the three child apps will be auto created..

Child Applications:

- live in namespace: argocd
  Actual workloads:
- go to their own destination namespaces

🧠 Important Notes:

- Parent app does NOT deploy workloads directly
- Parent app only creates child Applications
- Child apps deploy actual Kubernetes resources
- Child Application objects live in "argocd" namespace
- Actual apps run in their own destination namespaces

## 3) Mulicluster Environment:

Now lets take a cse of real world, where we need to run 3 diff clusters - one for dev, one for staging and another for prod..

So accordingly the steps will be:

=> you will have 3 apps with each having its deployment : dev-app, stag-app, prod-app
-> for now, lets have as above - one app of online-shop, one for nginx, and another for apache..

ArgoCD can manage multiple Kubernetes clusters from a single control plane.

### Steps:

🔹Step 1:
Step A: Create Kubernetes clusters

```
kind create cluster --name dev
kind create cluster --name staging
kind create cluster --name prod
```

Step B: Make sure kubectl knows them as contexts

```
kubectl config get-contexts
```

Step C: Add/register those contexts in ArgoCD

```
argocd cluster add kind-dev
argocd cluster add kind-staging
argocd cluster add kind-prod
```

🔹 Step 2: ArgoCD now knows multiple clusters

```
argocd cluster list
```

🔹 Step 3: Application decides WHERE to deploy
This is the key:

```
destination:
    server: <cluster-api-endpoint>
    namespace: dev
```

🧠 VERY IMPORTANT CONCEPT : Application controls cluster selection using "destination.server"

Example: same app → different clusters:
🔹 DEV app:

```YAML
metadata:
name: nginx-dev

spec:
source:
    repoURL: <repo>
    path: nginx

destination:
    server: https://dev-cluster-api
    namespace: dev
```

🔹 STAGING app:

```YAML
metadata:
name: nginx-staging

spec:
source:
    repoURL: <repo>
    path: nginx

destination:
    server: https://staging-cluster-api
    namespace: staging
```

🔹 PROD app:

```YAML
metadata:
name: nginx-prod

spec:
source:
    repoURL: <repo>
    path: nginx

destination:
    server: https://prod-cluster-api
    namespace: prod
```

### 🔥 Problem with this approach:

Now you see: Same app repeated 3 times
👉 duplication
👉 hard to scale
👉 error-prone

### 🚀 Now combine with App of Apps

Instead of managing all these manually:
You create: Parent app → manages all environments
So:
App of Apps = grouping/management hierarchy
Multi-cluster = deployment target choice

Example: Same-cluster case

Now suppose company policy says:
dev must run in dev cluster
staging must run in staging cluster
prod must run in prod cluster

Now the child apps may still look like:
root-app
→ nginx-dev
→ nginx-staging
→ nginx-prod
But this time each child app uses a different destination.server. So now they are still apps, yes — but each app points to a different cluster.

### App of Apps and Multi-cluster are not the same thing:

App of Apps answers: “How do I organize many Applications?”
Multi-cluster answers: “Which cluster does each Application deploy to?”

And then:
You can use App of Apps without multi-cluster - if all apps deploy into the same cluster.
You use App of Apps + multi-cluster together - when child apps must go to different clusters.

### 🧠 BUT STILL PROBLEM EXISTS:

You are still writing multiple YAML files manually. THIS leads to next concept
👉 ApplicationSet
🧠 Why ApplicationSet is needed:
ApplicationSet = automatically generate multiple Applications using templates

## 4) ApplicationSet:

This automatically generates multiple ArgoCD Applications from a template.
🧠 Simple meaning:
Instead of: Write 10 Application YAMLs manually
You do: Write ONE template → ArgoCD creates 10 Applications automatically
🔥 Core idea (VERY IMPORTANT):
ApplicationSet does NOT deploy workloads.
ApplicationSet creates Applications -> Applications deploy workloads

It is a Kubernetes Custom Resource (CR), Just like: kind: Application, Now we have: kind: ApplicationSet

🧪 First basic example (very simple): ApplicationSet YAML

```YAML

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet    ###****
metadata:
    name: nginx-multi-env
    namespace: argocd
spec:
    generators:  ####*****
        - list:
            elements:
            - name: dev
                namespace: dev
            - name: staging
                namespace: staging
            - name: prod
                namespace: prod
    template:
        metadata:
            name: nginx-{{name}}   ####*******
        spec:
            project: default
            source:
                repoURL: https://github.com/<your-username>/<repo>.git
                targetRevision: main
                path: nginx
            destination:
                server: https://kubernetes.default.svc
                namespace: {{namespace}}
            syncPolicy:
                automated:
                    prune: true
                    selfHeal: true
```

Another Exapmle : Compare them from generators.. there are different ways and types of generators:

```YAML
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet  ###****
metadata:
    name: nginx-multi-env
    namespace: argocd
spec:
    generators:  ####*****
        - list:
            elements:
            - app: nginx
                path: ui_approach/nginx
                namespace: dev
            - app: online-shop
                path: multicluster/online-shop
                namespace: staging
            - app: chaiapp
                path: applicationsets/chai-app
                namepspace: prod
    template:
        metadata:
            name: '{{app}}-list' ####****
        spec:
            project: default
            source:
                repoURL: https://github.com/<your-username>/<repo>.git
                targetRevision: main
                path: {{path}}  ####******
            destination:
                server: https://kubernetes.default.svc
                namespace: {{namespace}}  ####****
            syncPolicy:
                automated:
                    prune: true
                    selfHeal: true
```

### 🧠 1) First — What is a Generator?

Generator = Source of data used to create multiple Applications.
🧠 Simple meaning:

Generator answers:
“How many apps to create?”
“What variations to create?”

ApplicationSet = template + generator
template = how app should look
generator = how many apps + variations

### 🧩 2) Types of Generators (IMPORTANT)

1️⃣ List Generator (you already saw)
2️⃣ Git Generator
3️⃣ Cluster Generator

We will go one by one — slowly.

🟢 LIST GENERATOR:
You already saw basic version above.... List generator = manually define a list of values.
👉 You are manually writing the data 👉 This is static

🟡 GIT GENERATOR (VERY IMPORTANT):
🔥 Problem it solves:
Instead of writing: dev, staging, prod => 👉 It reads from Git automatically
Git Generator = Generate Applications based on files/folders in Git

🔹 Case 1: Directory-based generator

Example repo:
Suppose we have:

```
    repo/
        apps/
            dev/
            staging/
            prod/
```

```YAML:
generators:
    - git:
        repoURL: https://github.com/user/repo.git
        revision: main
        directories:
            - path: apps/*
template:
    metadata:
        name: app-{{path.basename}}
    spec:
        source:
            repoURL: https://github.com/user/repo.git
            targetRevision: main
            path: apps/{{path.basename}}
        destination:
            server: https://kubernetes.default.svc
            namespace: {{path.basename}}
```

🧠 What happens here:
Git folders:
apps/dev
apps/staging
apps/prod

👉 become:
app-dev
app-staging
app-prod
Folder structure = Application structure

🔹 Case 2: File-based generator:

Example repo:

```
    configs/
        dev.yaml
        staging.yaml
        prod.yaml
```

```YAML:
generators:
    - git:
        repoURL: https://github.com/user/repo.git
        revision: main
        files:
            - path: configs/*.yaml
```

🧠 Meaning : Each file = one Application

🧠 When to use Git generator:
Use when:
dynamic environments
Git-driven structure
team-based deployment
large projects

🔴 CLUSTER GENERATOR (VERY IMPORTANT):
Cluster Generator = Generate Applications for each registered cluster. 🔥 This is multi-cluster automation
Instead of:
dev-cluster
staging-cluster
prod-cluster
👉 It reads from: argocd cluster list

```YAML:
generators:
    - clusters: {}
template:
    metadata:
        name: app-{{name}}
    spec:
        destination:
            server: {{server}}
            namespace: default
```

🧠 Output:
👉 ArgoCD creates:
app-dev-cluster
app-staging-cluster
app-prod-cluster

🧠 COMBINATION (VERY POWERFUL): Combine Git + Cluster

Example: 3 clusters × 3 environments = 9 apps

```YAML:
generators:
    - matrix:
        generators:
            - clusters: {}
            - git:
                repoURL: https://github.com/user/repo.git
                revision: main
                directories:
                - path: apps/*
```

🧠 Output:
=>
dev-cluster + dev
dev-cluster + staging
dev-cluster + prod
=>
staging-cluster + dev
staging-cluster + staging
staging-cluster + prod
=>
prod-cluster + dev
prod-cluster + staging
prod-cluster + prod

# CONFIGURATION MANAGEMENT: HEML & KUSTOMIZE:

## Helm Intergartion:

ArgoCD Helm way:
Git → ArgoCD → Helm rendering → cluster

What problem Helm solves inside ArgoCD?
Without Helm, if you want 3 environments: dev, staging, prod=>
you may end up writing many similar YAML files again and again.
Helm solves that by giving: templates = reusable blueprint, values = environment-specific input

You already know this partly, but let us connect it to ArgoCD properly.
A simple Helm chart usually has:

```
myapp-chart/
    Chart.yaml
    values.yaml
    templates/
        deployment.yaml
        service.yaml
        ingress.yaml
```

Now in ArgoCD, when source.path points to this folder, ArgoCD understands:
“this is a Helm chart source” => Then it uses Helm rendering on that chart.

### First simple Helm Application in ArgoCD:

```YAML

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: nginx-helm-app
namespace: argocd
spec:
project: default
source:
    repoURL: https://github.com/your-username/your-repo.git
    targetRevision: main
    path: helm/nginx-chart
destination:
    server: https://kubernetes.default.svc
    namespace: dev
syncPolicy:
    automated:
    prune: true
    selfHeal: true

```

What happens here internally:
Step by step:
=> Kubernetes receives this Application
=> ArgoCD watches it
=> Repo server clones Git repo
=> It sees Chart.yaml in helm/nginx-chart
=> It uses Helm to render templates into final YAML
=> Application controller compares rendered YAML with cluster live state
=> If different, ArgoCD syncs them
This is the same Application flow you already noted, but now the source type is Helm instead of plain YAML.

Where values.yaml fits:
=> Suppose values.yaml contains:

```YAML

replicaCount: 2
image:
    repository: nginx
    tag: "1.27"
service:
    type: ClusterIP
    port: 80
```

=> and your templates/deployment.yaml uses:

```YAML
replicas: {{ .Values.replicaCount }}
```

=> Then ArgoCD reads those values while rendering.

### Default values vs override values:

Usually a chart has:
=>one common values.yaml
=> optional extra values files like:
values-dev.yaml
values-staging.yaml
values-prod.yaml
Common things stay in values.yaml. Environment-specific things go in override files.

Example:
=>values.yaml:

```YAML
image:
    repository: myapp
service:
    port: 80
```

=>values-dev.yaml:

```YAML
replicaCount: 1
image:
    tag: dev-latest
```

=>values-prod.yaml

```YAML
replicaCount: 3
image:
    tag: prod-v1
```

#### How ArgoCD uses a specific values file:

Now we extend the Application YAML-

```YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: nginx-dev
namespace: argocd
spec:
project: default
source:
    repoURL: https://github.com/your-username/your-repo.git
    targetRevision: main
    path: helm/nginx-chart    ####*****
    helm:                     ####*****
        valueFiles:
            - values-dev.yaml
destination:
    server: https://kubernetes.default.svc
    namespace: dev
syncPolicy:
    automated:
    prune: true
    selfHeal: true
```

In the above the key new block is:

```YAML
helm:
    valueFiles:
        - values-dev.yaml
```

Meaning: “Render this Helm chart, but use values-dev.yaml as override input.”

When ArgoCD renders this chart:
=> it reads values.yaml first
=> then reads values-dev.yaml
=> the second file overrides matching keys from the first

So if: values.yaml says replicaCount = 2 and values-dev.yaml says replicaCount = 1 final rendered result becomes: replicaCount = 1
That is Helm override behavior.

### Connect this with your current learning chain:

Now Helm integration fits like this:
Application + Helm = one app, chart-based deployment
App of Apps + Helm = parent manages many Helm-based child Applications
ApplicationSet + Helm = generate many Helm-based Applications automatically

### Real repo structure for Helm + ArgoCD (Application):

=> For Learning: Lets say you have a git Repo Structure:

```
repo/
    helm/
        nginx-chart/
        Chart.yaml
        values.yaml
        values-dev.yaml
        values-staging.yaml
        values-prod.yaml
        templates/
            deployment.yaml
            service.yaml
```

=> Dev, staging, prod with 3 separate Applications

#### dev app:

```YAML

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: nginx-dev
namespace: argocd
spec:
    project: default
    source:
        repoURL: https://github.com/your-username/your-repo.git
        targetRevision: main
        path: helm/nginx-chart
        helm:
            valueFiles:
                - values-dev.yaml
    destination:
        server: https://kubernetes.default.svc
        namespace: dev
```

#### staging app:

```YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: nginx-staging
namespace: argocd
spec:
    project: default
    source:
        repoURL: https://github.com/your-username/your-repo.git
        targetRevision: main
        path: helm/nginx-chart
        helm:
            valueFiles:
                - values-staging.yaml
    destination:
        server: https://kubernetes.default.svc
        namespace: staging
```

#### prod app:

```YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: nginx-prod
namespace: argocd
spec:
    project: default
    source:
        repoURL: https://github.com/your-username/your-repo.git
        targetRevision: main
        path: helm/nginx-chart
        helm:
            valueFiles:
                - values-prod.yaml
    destination:
        server: https://kubernetes.default.svc
        namespace: prod
```

Now you can already see the next problem: the YAML is again repetitive. And that takes us naturally to ApplicationSet + Helm.

### Real repo structure for Helm + ArgoCD (ApplicationSet):

Now instead of writing 3 Application YAMLs manually, we generate them.

```YAML
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
    name: nginx-helm-multi-env
    namespace: argocd
spec:
    generators:
        - list:
            elements:
                - env: dev
                namespace: dev
                valuesFile: values-dev.yaml
                - env: staging
                namespace: staging
                valuesFile: values-staging.yaml
                - env: prod
                namespace: prod
                valuesFile: values-prod.yaml
    template:
        metadata:
            name: 'nginx-{{env}}'
        spec:
            project: default
            source:
                repoURL: https://github.com/your-username/your-repo.git
                targetRevision: main
                path: helm/nginx-chart
                helm:
                    valueFiles:
                        - '{{valuesFile}}'
            destination:
                server: https://kubernetes.default.svc
                namespace: '{{namespace}}'
            syncPolicy:
                automated:
                    prune: true
                    selfHeal: true
```

### 🔥 HELM BLOCK INSIDE ARGOCD (source.helm):

helm.valueFiles (we already saw this), helm.parameters, helm.values, helm.releaseName

🧩 Full Helm block (all important fields)

```YAML
helm:
    valueFiles:
        - values-dev.yaml
    parameters:
        - name: image.tag
            value: v2
    values: |
        replicaCount: 3
    releaseName: my-nginx
    version: v3
```

#### valueFiles:

🧠 Meaning: Use this file as override input.
🔁 Flow:
-> Read values.yaml (default)
=> Then apply values-dev.yaml
=> Override matching fields
📌 Example:

```YAML
values.yaml:
    replicaCount: 2
values-dev.yaml:
    replicaCount: 1
```

👉 Final result = replicaCount: 1

#### parameters:

🧠 What is this?:
This is equivalent to: helm install myapp ./chart --set image.tag=v2
🧠 Meaning: Override a specific key directly
🔥 When to use parameters: Use when -> You want small override, You don't want to create a separate values file and Quick change needed

#### values:

🧠 Meaning: Write values directly inside Application YAML
📌 Equivalent to: helm install myapp ./chart -f inline-values.yaml
🔥 Use case: small testing configs

#### releaseName:

```YAML
helm:
    releaseName: my-nginx
```

🧠 What is release name?
In Helm CLI: helm install my-nginx ./chart 👉 my-nginx = release name
🧠 In ArgoCD: If not given: 👉 ArgoCD uses Application name
Example:

```YAML
    metadata:
        name: nginx-dev
```

👉 default release name = nginx-dev
🔥 Why this matters:
Because Kubernetes resources may include:
name: {{ .Release.Name }}-service
So: releaseName changes resource naming.

🧠 Priority (TOP → BOTTOM):

1. parameters (highest)
2. values (inline)
3. valueFiles
4. values.yaml (default)
   🔥 Example:
   values.yaml → replicaCount: 2
   values-dev.yaml → replicaCount: 3
   values (inline) → replicaCount: 4
   parameters → replicaCount: 5
   👉 FINAL RESULT: replicaCount = 5

🔥 Best Practice:

- Use valueFiles for env configs
- Avoid large inline values
- Use ApplicationSet for scaling

# NOTIFICATIONS: PART A- SLACK

When you make changes to your deployment, if you want notifications in your email / slack notifications, then you can use this...

## 🧠 1) First — What is “Slack”?

👉 Slack

### 🔹 Simple meaning:

Slack is a team communication tool used in companies.

### 🔹 Think like this:

Instead of:

- emails ❌ (slow, messy)
- WhatsApp ❌ (not professional)

Companies use:

👉 Slack = organized messaging for teams

### 🔹 Example:

In a company, you may have:

- #dev-team
- #backend
- #frontend
- #alerts
- #production-issues

These are called **channels**.

### 🔹 What happens in Slack?

People:

- chat with team members
- share files
- discuss bugs
- get system alerts (VERY IMPORTANT for DevOps)

### 🔥 Why Slack is important for DevOps

Because systems send messages like:

- deployment succeeded ✅
- deployment failed ❌
- app crashed 🚨
- new version released 📦

👉 Instead of checking manually, you get **instant alerts**

---

## 🧠 2) Now — What are Notifications in ArgoCD?

### 🔹 Definition

Notifications =
👉 “Automatically inform users when something happens in ArgoCD”

---

### 🔹 What kind of events?

ArgoCD can notify when:

- Application synced ✅
- Sync failed ❌
- Health degraded 🚨
- New deployment triggered 📦
- Rollback happened 🔁

---

### 🔹 Where can notifications go?

- Email 📧
- Slack 💬
- Webhooks 🌐
- Microsoft Teams

---

### 🔥 So the idea is:

ArgoCD event → Notification system → Slack/Email

---

## 🧠 3) Why Notifications are needed (VERY IMPORTANT)

### Without notifications:

- You don’t know if deployment failed
- You don’t know if app is broken
- You must manually check UI

---

### With notifications:

- instant alerts
- faster debugging
- production safety

---

### 🎯 Real-world example

You deploy new version of EMS app:

👉 If something breaks:
❌ Deployment FAILED: backend-prod

👉 Immediately team reacts

---

## 🧠 4) How ArgoCD Notifications work (Architecture)

### 🔹 Component

ArgoCD has a feature called:

👉 ArgoCD Notifications Controller

---

### 🔁 Flow:

ArgoCD detects event
↓
Notification controller triggers
↓
Message sent to Slack / Email

---

## 🧠 5) Setup Concept (before YAML)

To enable notifications, we need:

### STEPS:

🔹 Step 1: Install notifications

If using Helm:

```
helm upgrade --install argocd argo/argo-cd \
--set notifications.enabled=true
```

🔹 Step 2: Configure destination (Slack / Email)

👉 This is done using:

ConfigMap
Secret

---

## 🧩 6) Slack Notification Setup (STEP-BY-STEP):

🔹 Step A: Create Slack Webhook
In Slack:
go to “Apps”
search: Incoming Webhooks
create webhook
copy URL

Example:
https://hooks.slack.com/services/XXXX

🔹 Step B: Store webhook in Kubernetes Secret

```YAML

apiVersion: v1
kind: Secret
metadata:
    name: argocd-notifications-secret
    namespace: argocd
stringData:
    slack-token: https://hooks.slack.com/services/XXXX

```

🔹 Step C: Configure Notification ConfigMap

```YAML

apiVersion: v1
kind: ConfigMap
metadata:
    name: argocd-notifications-cm
    namespace: argocd
data:

    service.slack: |
        token: $slack-token

    template.app-sync-succeeded: |
        message: "✅ App {{.app.metadata.name}} synced successfully"

    trigger.on-sync-succeeded: |
        - when: app.status.operationState.phase == 'Succeeded'
        send: [app-sync-succeeded]

```

## 🧠 7) Connecting Notification to Application

Now we must tell which app should send notification.
Add annotation in Application:

```YAML
            metadata:
                annotations:
                    notifications.argoproj.io/subscribe.on-sync-succeeded.slack: my-channel
```

🔥 Meaning: 👉 “Send Slack message to this channel when sync succeeds”

# # NOTIFICATIONS: PART B- EMAIL

## 🧠 1) First — How Email Notification Works (Concept)

Flow is same as Slack, only destination changes: ArgoCD event → Notification Controller → Email (SMTP server)
👉 Instead of Slack webhook, we use: 👉 **SMTP server (mail server)**

---

## 🧠 2) What is SMTP? (VERY IMPORTANT)

SMTP = Simple Mail Transfer Protocol

👉 It is the protocol used to send emails.

---

### 🔹 For Gmail:

SMTP Server = `smtp.gmail.com`
Port = `587`

---

## 🧠 3) What do we need for Gmail setup?

To send email from ArgoCD, we need:

- Gmail email ID
- App Password (NOT your normal password)
- SMTP server details

---

## 🔐 4) VERY IMPORTANT — Gmail App Password

❗ Gmail does NOT allow normal password for apps

👉 You MUST create **App Password**

---

### 🔹 Steps:

1. Go to Google Account
2. Enable **2-Step Verification**
3. Go to **App Passwords**
4. Generate password for “Mail”
5. Copy it

Example: abcd efgh ijkl mnop

👉 This is what we will use in Kubernetes Secret

---

## 🧩 5) Step-by-Step Setup

---

🔹 Step A: Create Secret (STORE EMAIL + PASSWORD)

```YAML

apiVersion: v1
kind: Secret
metadata:
    name: argocd-notifications-secret
    namespace: argocd
stringData:
    email-username: your-email@gmail.com
    email-password: your-app-password

```

🔹 Step B: Configure ConfigMap (EMAIL SERVICE)

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
    name: argocd-notifications-cm
    namespace: argocd

data:
    service.email: |
        host: smtp.gmail.com
        port: 587
        username: $email-username
        password: $email-password
        from: your-email@gmail.com

    template.app-sync-succeeded: |
        subject: "App {{.app.metadata.name}} Sync Status"
        message: |
        ✅ Application {{.app.metadata.name}} synced successfully.

    trigger.on-sync-succeeded: |
        - when: app.status.operationState.phase == 'Succeeded'
        send: [app-sync-succeeded]

```

---

## 🧠 6) Connect Email to Application

Now we must tell: 👉 which email should receive notification
Add annotation:

```YAML
metadata:
    annotations:
        notifications.argoproj.io/subscribe.on-sync-succeeded.email: your-email@gmail.com
```

---

# IMAGE UPDATER:

## 🧠 1) What is Image Updater? (Concept)

👉 ArgoCD Image Updater =
A tool that automatically updates container image versions in your Git repository.

In a professional environment, we should not change the app yaml file again and again from git hub.
It should rather be realesed in vesrions..

So, ARGOCD provides a service called image-updater, that automatically detects new vesrsions in the container image registry and updates it in the git hub.
So, no manual updation is required..

Example registries:
Docker Hub
AWS ECR
GitHub Container Registry
Google Artifact Registry

Argo CD Image Updater is an add-on tool that can update container image versions for Argo CD applications and supports writing changes back to Git.

---

## 🔹 Problem it solves

Without Image Updater:

1. You build a new Docker image (e.g., `v2`)
2. You manually go to Git
3. Update image tag in:
   - values.yaml OR
   - Application YAML
4. Commit & push
5. ArgoCD syncs

With Image Updater:
New image pushed → Image Updater detects → Updates Git → ArgoCD syncs

👉 No manual Git changes needed

---

## 🧠 2) VERY IMPORTANT CORRECTION (Understand Properly)

👉 Image Updater checks **container registry**, NOT cluster

Examples:

- Docker Hub
- :contentReference[oaicite:0]{index=0}
- :contentReference[oaicite:1]{index=1}

---

## 🔥 Correct flow:

CI pushes new image → Registry updated
->
Image Updater checks registry
->
Updates Git (values.yaml / parameters)
->
ArgoCD detects change
->
Deploys new version

## 🧠 3) Where does Image Updater fit?

CI Pipeline
->
Build Docker Image
->
Push to Registry
->
Image Updater
->
Update Git
->
ArgoCD
->
Deploy to Kubernetes

---

## 🧠 4) How Image Updater works (Internally)

Step-by-step:

1. Watches container registry
2. Checks for new tags
3. Matches rules (latest / semver / regex)
4. Updates Git repo (or Helm values)
5. Commits changes
6. ArgoCD detects change
7. Sync happens

---

## 🧠 5) How Image Updater knows which image to track

👉 Using annotations in Application

🔹 Example Application with Image Updater

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: backend-app
namespace: argocd

annotations:
argocd-image-updater.argoproj.io/image-list: backend=myrepo/backend
argocd-image-updater.argoproj.io/backend.update-strategy: latest

spec:
project: default

source:
  repoURL: https://github.com/your-repo.git
  targetRevision: main
  path: apps/backend

helm:
  valueFiles:
    - values.yaml

destination:
  server: https://kubernetes.default.svc
  namespace: dev
```

🔹 image-list
argocd-image-updater.argoproj.io/image-list: backend=myrepo/backend ; 👉 Meaning: Track image: myrepo/backend
🔹 update-strategy
argocd-image-updater.argoproj.io/backend.update-strategy: latest
👉 Meaning: Use latest available image tag

---

## 🧠 7) Update Strategies (VERY IMPORTANT)

🔹 latest
👉 Always pick newest tag -> v1 → v2 → v3 → always picks v3
🔹 semver
👉 Follow version rules -> 1.0.0 → 1.1.0 → 1.2.0 (allowed) || 1.0.0 → 2.0.0 (may not be allowed)

---

# LOGS AND MONITORING: (Prometheus & Grafana) (**_VVIMP_**):

## Prometheous:

### What is Prometheous?
