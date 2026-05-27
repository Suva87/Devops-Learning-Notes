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
kubectl get secret argocd-initial-admin-secret -n argocd \ -o jsonpath="{.data.password}" | base64 -d && echo
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
argocd cluster list
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
4.  ArgoCD generates manifests
5.  ArgoCD compares with cluster
6.  If OutOfSync → sync (auto/manual)

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

## 🧠 8) Steps to use Image Updater:

### 1. Install Image Updater:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argo-image-updater/stable/manifests/install.yaml
```

### 2. Verify that POD is running:

```
kubectl -n argocd get pods -l app.kubernetes.io/name=argocd-image-updater
```

### 3. Login tp Dockerhub: as image updater always looks at dockerhub

For this go to your dockerhub->account settings->personal access tokens->generate token->copy the login and password

```
docker login ******
```

### 4. Pull the base Image from dockerhub

```
docker pull <docker_user_name>/<app_name>:latest
```

### 5. If the docker image is not yours then you can tag it in your dockerhub:

```
docker tag <docker_user_name>/<app_name>:latest sanyal_suva/<app_name>:v1.0.0
docker push sanyal_suva/<app_name>:v1.0.0
```

#### KUSTOMIZE: kustomization.yaml

we are learning this as we will be using this as an example for image updater. We require kustomize for using image updater..

this is a service of K8S.

similar to deployment.yaml / satefulset.yaml we must create kustomization.yaml in the same directory...

```YAML
    apiVesrion: kustomize.config.k8s.io/v1betal
    kind: kustomization
    #list of manifest files in this directory (base yml files)
    resources:
        - deployment.yaml
        - service.yaml
        - secret.yaml
# (optional: you can put commonLabels or namespace here)
# comminLabels:
#       app: chai-app
```

### 6. GIT HUB LOGIN:

Go to github->settings->developer settings->personal access token->classic->generate new token
copy the pass key somewhere as it disaapears after closing the tab

### 7. Use gut credentials in secret.yaml

edit the current secret.yaml

```YAML
apiVersion: v1
kind: secret
metadata:
    name: argocd-image-updater-git-creds
    namespace: argocd
stringData:
    username: "<github_username>"
    password: "personal_access_token"
```

### 8. Annotate your application as mentioned in notes above:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-app
  namespace: argocd

annotations:
  # maps the alias "your app" to your image
  argocd-image-updater.argoproj.io/image-list: <app_name>=<your-dockerhub-username>/<image-name>

  #Use git write back
  argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/argocd-image-updater-git-creds

  # Update strategy: use semantic versioning
  # semver format: MAJOR.MINOR.PATCH
  # MAJOR-> BREAKING CHANGES ( eg. 1.X.X -> 2.0.0)
  # MINOR -> NEW FEATURES (eg. 1.1.0 -> 1.2.0)
  # PATCH -> BUG FIXES (eg. 1.1.1 -> 1.1.2)
  argocd-image-updater.argoproj.io/backend.update-strategy: semver #

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

  syncPloicy:
    automated:
      prune: true
      selfHeal: true
```

Now whenever you push a new version to your docker hub, let say <your-dockerhub-username><iamge-name>:v1.0.1 you can tail the logs and seee..

```
kubectl -n argocd logs deploy/argocd-image-updater -f
```

# MONITORING:

You already learned Prometheus + Grafana in Kubernetes

## What EXTRA is needed for ArgoCD Monitoring?

ArgoCD does not replace monitoring, it changes how monitoring components themselves are deployed and managed.

In Kubernetes notes you learned:

```
Helm install kube-prometheus-stack
```

and then manually:
-> create ServiceMonitor
-> create PrometheusRule
-> create Grafana dashboards

But now we are in GitOps world: So the question becomes: "How do we GitOps-manage Prometheus, Grafana, ServiceMonitor, PrometheusRule, dashboards?"

## 🧠 What extra components become important?

You already know:
✅ ServiceMonitor
✅ PrometheusRule
✅ Dashboard ConfigMap
✅ kube-prometheus-stack Helm chart
Now ArgoCD adds deployment management layer

## 🚀 New Things for ArgoCD Monitoring

You need to learn:
✅Monitoring stack as Helm application
✅Monitoring stack inside App of Apps
✅Monitoring stack inside ApplicationSet
✅GitOps dashboards
✅GitOps alerts
✅ArgoCD self-monitoring (VERY IMPORTANT)

## 🔥 ARGOCD SELF-MONITORING

ArgoCD exposes metrics itself. Meaning ArgoCD has: /metrics, just like your NodeJS app.

ArgoCD components expose metrics:

| Component                      |       Example metrics |
| ------------------------------ | --------------------: |
| argocd-server                  |         request count |
| argocd-repo-server             |        Git operations |
| argocd-application-controller  |            sync count |
| argocd-notification-controller | notification failures |

Prometheus can collect:

1.

```
argocd_app_sync_total
```

Meaning: How many syncs happened

2.

```
argocd_app_health_status
```

Meaning: How many applications unhealthy

3.

```
argocd_app_reconcile
```

Meaning: How often reconciliation occurs

### Example ServiceMonitor for ArgoCD

```YAML
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-monitor
  namespace: monitoring
  labels:
    release: prometheus-stack

spec:
  namespaceSelector:
    matchNames:
      - argocd

  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server

  endpoints:
    - port: metrics
      interval: 15s
```

🧠 What happens here?

Prometheus says:

```
Find Services:

app.kubernetes.io/name=argocd-server

Go to metrics endpoint

Scrape every 15 seconds
```

🧠 GitOps Dashboard Flow:

```
Dashboard JSON
      ↓
ConfigMap
      ↓
Git
      ↓
ArgoCD
      ↓
Grafana auto-import
```

Example folder structure:

```
monitoring/

    prometheus/

        servicemonitors/
            ems-api-monitor.yaml
            argocd-monitor.yaml

        rules/
            ems-alerts.yaml
            argocd-alerts.yaml

        dashboards/
            ems-dashboard.yaml
            argocd-dashboard.yaml
```

## STEPS TO SET UP MONITORING OF APPLICATION: (but through helm)

1. Deploy all the apps in the respective namespaces...by using argoCD and helm

2. Check the port of ArgoCd metrics and ArgoCD server metrics

```
kubectl get svc -n argocd
```

3. Install Prometheus:

```
helm repo add prometheus-community.github.io/helm-charts
kubectl create ns monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

You will see that it will show you grafana username and also the way you can get the password..

```
kubectl -n monitoring get secrets kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

4. Create Service Monitor:
   We will be picking three services now for prometheus to scrape the data from:
   vim metrics.yaml

```YAML
# this is for argocd-metrics
apiVesrion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
    name: argocd-metrics
    namespace: argocd
    labels:
        release: kube-prometheus-stack
spec:
    selector:
        matchLabels:
            app.kubernetes.io/name: argocd-metrics
    endpoints:
    - port: metrics
---

# this is for argocd-server-metrics
apiVesrion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
    name: argocd-server-metrics ###
    namespace: argocd
    labels:
        release: kube-prometheus-stack
spec:
    selector:
        matchLabels:
            app.kubernetes.io/name: argocd-server-metrics ###
    endpoints:
    - port: metrics
---

# this is for argocd-repo-server-metrics
apiVesrion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
    name: argocd-repo-server-metrics ###
    namespace: argocd
    labels:
        release: kube-prometheus-stack
spec:
    selector:
        matchLabels:
            app.kubernetes.io/name: argocd-repo-server  ###
    endpoints:
    - port: metrics
---

```

5. get the ports for prometheus and grafana

```
kubectl get svc -n monitoring
```

6. port forward

```
kubectl port-forward svc/<check-svc-name-from-previous-command-for-prometheus> -n monitoring 9090:<port-no-of-svc> --address=0.0.0.0 &

kubectl port-forward svc/<check-svc-name-from-previous-command-for-grafana> -n monitoring 3000:<port-no-of-svc> --address=0.0.0.0 &
```

7. Make sure you add the ports in EC2..

8. Now open the grafana from ec2 public-ip/3000

9. login with the previously gathered username & password

10. inside grafana:
    a) connections->data source-> dashboard (this is the default dashboard)

b) to make customisable dashboard: ->

i)) now there are two ways:
=> you can either add visualisation manually:
-> dashboard -> new dashboard
-> add visualisation-> select the data source as prometheus
-> metrics -> select "argocs_app_info"
-> select labels -> "project" => "seelect the project you app is in"
-> operations -> select what you like
-> run query
-> now in sidebar you can see sugestions..

ii)) the previous method is very slow.. so lets import dashboard
=> dashboard -> import
-> click on the link "Find and import dashboard"
-> in the serach bar search for "argoCD"
-> select the appropriate dashboad (there is one very good one -> 19993)
-> copy id to clipboard
-> go back to the import page of grafana -> paste the id -> click on Load -> on new page click "import"

# RBAC (Role based Access Control):

In AargoCd there are 2 types of users:
Admin User | Local User

Admin user generally has all the access, however local user may have CLI access or UI access or both..

In ArgoCD, access can come from:

- built-in admin user
- local users
- SSO users/groups, such as GitHub, Google, Azure AD, Keycloak, etc.

We generally have a configMap.yaml to define the roles and access for a user..

Eaxmple: argo-local-user-cm.yaml

a)) create local user

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
    name: argocd-cm
    namespace: argocd
    labels:
        app.kubernetes.io/name: argocd-cm
        app.kubernetes.io/part-of: argocd
    data:
        accounts.alice: apiKey, login # Can generate tokens and login to UI
        accounts.jhon: login # Can only login to UI
```

After applying this you will see that in the UI, if you are looged in as admin, alice and john use has been created..

b)) create their password:

```
argocd account update-password --account alice
```

=> now enter password for admin -> then enter password for user
=> now the user 'alice' should be able to login with the password in the UI

###### FROM HERE ON CONTINUE WITH THE NOTES ON GITHUB.. ARGOCD IN ONE SHOT.... CHAPTER 09_SECURITY_SCALING - before going on to that note, please learn https method from below-

---

# SECURITY CERTIFICATES (HTTPS) - FOR REAL WORLD HOSTING

## 1) First — what are we trying to achieve?

Right now, in your learning setup, you accessed ArgoCD like this:

```
kubectl port-forward service/argocd-server -n argocd 8080:443 --address=0.0.0.0
```

But in real-world EKS hosting, we do not want this. We want: https://argocd.yourdomain.com

So final goal:

```
Browser
  ↓
https://argocd.testdomain.com
  ↓
DNS
  ↓
AWS Application Load Balancer
  ↓
Kubernetes Ingress
  ↓
argocd-server Service
  ↓
ArgoCD UI
```

## 2) What components are needed?

For HTTPS ArgoCD on EKS, we need these components:

```
1. EKS cluster
2. ArgoCD installed using Helm
3. AWS Load Balancer Controller
4. ACM certificate
5. Ingress resource
6. DNS record
```

Now let us understand each one slowly.

1. EKS Cluster:

EKS is AWS-managed Kubernetes. In your local learning, you used kind. In real AWS production, you use: Amazon EKS
So instead of: "kind create cluster" ... You will eventually do something like:

```
eksctl create cluster ...
```

2. ArgoCD installed using Helm

```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create ns argocd
helm install argocd argo/argo-cd -n argocd
```

For HTTPS, we will not expose ArgoCD by port-forward anymore.
Instead, we will expose it through: Ingress → AWS ALB → HTTPS certificate

3. AWS Load Balancer Controller

a)) What is AWS Load Balancer Controller?
AWS Load Balancer Controller is a Kubernetes controller that watches Kubernetes resources like Ingress and then creates AWS load balancers for them.
So when you create an Ingress like: "kind: Ingress", The AWS Load Balancer Controller sees it and creates:
AWS Application Load Balancer
Target Groups
Listeners
Rules

4. ACM Certificate:

ACM means: AWS Certificate Manager. ACM gives you an SSL/TLS certificate. AWS Certificate Manager verifies that you own the domain before issuing the certificate. With DNS validation, ACM gives you a CNAME record to add to DNS.

5. Ingress Resource

Ingress is a Kubernetes object that says:
When traffic comes for this domain, send it to this Kubernetes service.

ArgoCD’s own ingress documentation explains that when TLS is terminated at the ingress layer, the ArgoCD API server is commonly run with TLS disabled behind the ingress, using server.insecure: "true" or the --insecure flag.

6. DNS Record

DNS means: Domain Name System
DNS connects your domain name to the AWS Load Balancer.

If your DNS is in Route 53, you create records there. AWS describes Route 53 as a service for domain registration, routing internet traffic to resources, and health checking.

If your domain is registered in GoDaddy or Cloudflare, you have two choices:

Option A:
Keep DNS in GoDaddy/Cloudflare
→ Add CNAME records there

Option B:
Move DNS management to Route 53
→ Update nameservers at GoDaddy/Cloudflare
→ Add records in Route 53

## The Real Architecture

```
User Browser
  ↓
https://argocd.testdomain.com
  ↓
DNS CNAME / Alias Record
  ↓
AWS Application Load Balancer
  ↓
HTTPS Listener : 443
  ↓
Target Group
  ↓
EKS Worker Node / Pod Target
  ↓
argocd-server Service
  ↓
argocd-server Pod
```

## First understand ALB, Listener, Target Group

1. What is ALB?
   ALB means: Application Load Balancer.
   Simple meaning: ALB is the public entry gate for your web traffic.
   For example:
   "https://argocd.testdomain.com" will first reach: "AWS Application Load Balancer". Then ALB forwards traffic into Kubernetes.

2. What is a Listener?
   A listener means: Which port is the load balancer listening on?

For real HTTPS:
Listener: 443
Certificate: ACM certificate

So the listener says: I am waiting for HTTPS traffic on port 443.

3. What is a Target Group?
   Target Group means: Where should the ALB send the traffic after receiving it?

Example:

```
ALB receives request
  ↓
Target Group decides backend targets
  ↓
Traffic goes to ArgoCD pods/service
```

Depending on configuration, targets can be: EC2 instance/node OR Pod IP

## Real-world flow in simple language

When someone opens: "https://argocd.testdomain.com", This happens:

```
1. Browser asks DNS:
   Where is argocd.testdomain.com?

2. DNS replies:
   Go to this AWS ALB.

3. Browser connects to ALB on port 443.

4. ALB uses ACM certificate to prove:
   Yes, this is a valid HTTPS site.

5. ALB checks Ingress routing rule:
   Host = argocd.testdomain.com

6. ALB forwards traffic to argocd-server.

7. ArgoCD UI opens securely.
```

## Steps to create https configuration on EKS:

### Before commands, understand the important assumption

We are assuming:
Domain: testdomain.com
ArgoCD URL: argocd.testdomain.com
AWS Region: ap-south-1
EKS cluster name: ems-eks-cluster
Namespace: argocd

=> GO TO EC2
-> Go to IAM -> Create a User -> Next-> "Attach Policies Directly" -> Select "AdminAccess" (Production should use least privilege) -> Create User
-> Go to "Security Credentials" -> "Create Access Key" -> CLI

=> in your Instance:
Install the follwoing:

### Prepare AWS CLI on your EC2/Linux machine

#### AWS CLI:

a)) If this is your first time updating on Amazon Linux, to install the latest version of the AWS CLI, you must uninstall the pre-installed yum version using the following command:

```
sudo yum remove awscli -y
```

b)) To install the AWS CLI, run the following commands:

please note that you should have zip installed in linux... to install it...

```
sudo yum install -y unzip
```

and then:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

c)) Check version:

```
aws --version
```

d)) Now configure AWS:

```
aws configure
```

It will ask:

```
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

e)) Then test:

```
aws sts get-caller-identity
```

Expected output:

```JSON
{
  "UserId": "...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/your-user"
}
```

If this returns your account ID, AWS CLI is working.

### Install kubectl, eksctl, Helm

You already know kubectl and Helm from your Kubernetes notes, but for EKS we also commonly use:eksctl

Why eksctl? eksctl helps create and manage EKS clusters.

Without eksctl, creating an EKS cluster manually requires setting up many things separately:
VPC
subnets
security groups
EKS control plane
node groups
IAM roles
OIDC provider
kubeconfig connection

#### Install eksctl on Amazon Linux / Linux EC2

Run these commands:

```bash
# For normal x86_64 / amd64 EC2 instances
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

# Download latest eksctl release
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# Optional but recommended: verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

# Extract eksctl
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

# Move eksctl binary to PATH
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl

### Connect kubectl to your EKS cluster
```

Verify installation:

```bash
eksctl version
```

After installing AWS CLI, kubectl, Helm, and eksctl, check all tools:

```bash
aws --version
kubectl version --client
helm version
eksctl version
```

### Create EKS cluster:

```bash
eksctl create cluster \
  --name ems-eks-cluster \
  --region ap-south-1 \
  --nodes 2 \
  --node-type t3.medium \
  --managed
```

This creates:
EKS control plane
worker nodes
node group
VPC networking
kubectl context

b)) Check:

```
kubectl get nodes
```

Expected:

```
NAME                                           STATUS   ROLES
ip-xxx-xxx-xxx-xxx.ap-south-1.compute.internal Ready
```

If this works, your terminal can talk to EKS.

### Install ArgoCD using Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### Create namespace:

```bash
kubectl create namespace argocd
```

### Now install ArgoCD.

But because we are planning to put ArgoCD behind AWS ALB HTTPS, we should set ArgoCD server as insecure internally.

#### Why insecure internally?

External traffic will be: Browser → HTTPS → ALB
Then inside the cluster: ALB → argocd-server
The ALB is terminating HTTPS.

Install ArgoCD with Helm:

```bash
helm install argocd argo/argo-cd \
  -n argocd \
  --set configs.params."server\.insecure"=true
```

Check pods:

```bash
kubectl get pods -n argocd
```

Check service:

```bash
kubectl get svc -n argocd
```

### Get ArgoCD initial admin password

```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

Later, change the password from UI.

### Install AWS Load Balancer Controller:

Now we need the component that will convert Kubernetes Ingress into AWS ALB.

1. What is AWS Load Balancer Controller?

Kubernetes has an object called: kind: Ingress. But EKS itself does not automatically know how to create an AWS ALB from that Ingress. So we install:
-----AWS Load Balancer Controller

But it runs inside Kubernetes as a pod.

Its job:

```
Watch Kubernetes Ingress
   ↓
Create AWS Application Load Balancer
   ↓
Create target groups
   ↓
Create listeners
   ↓
Connect ALB to Kubernetes services/pods
```

So the question is: How does a Kubernetes pod get permission to create AWS resources?
Answer: IAM Role for Service Account

2. Associate IAM OIDC provider:

A Kubernetes pod is not automatically trusted by AWS IAM. .. But the AWS Load Balancer Controller pod needs permission to call AWS APIs. So AWS needs a way to trust this Kubernetes service account. That is where OIDC comes in.

OIDC provider creates trust between: EKS cluster service accounts AND AWS IAM roles
The controller needs AWS permissions. In modern EKS, we give permissions using:

```
IRSA = IAM Role for Service Account
```

Meaning:

```
Kubernetes ServiceAccount
   ↓
linked to AWS IAM Role
   ↓
controller gets permission to create ALB
```

Run:

```bash
eksctl utils associate-iam-oidc-provider \
  --region $AWS_REGION \
  --cluster $CLUSTER_NAME \
  --approve
```

Meaning:

```
For this EKS cluster,
create/associate the OIDC provider in AWS IAM,
so Kubernetes service accounts can use IAM roles.
```

### Create IAM service account for AWS Load Balancer Controller

Now we create:

```
Kubernetes ServiceAccount
+
AWS IAM Role
+
IAM permissions
```

=> Download IAM policy:

```bash
curl -o iam_policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

=> Get your AWS account ID:

```bash
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo $AWS_ACCOUNT_ID
```

=> Create policy:

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

=> Save your AWS account ID:

```
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text) ***** ## how do i get the account_id?
```

### Create IAM service account:

=>

```bash
eksctl create iamserviceaccount \
  --cluster=$CLUSTER_NAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=$AWS_REGION
```

This creates:
ServiceAccount: aws-load-balancer-controller
Namespace: kube-system
IAM Role: AmazonEKSLoadBalancerControllerRole
Policy: AWSLoadBalancerControllerIAMPolicy

=> Check:

```bash
kubectl get sa aws-load-balancer-controller -n kube-system
```

### Install controller using Helm:

a)) Add repo:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

b)) Install:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$CLUSTER_NAME \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

Why these two lines?

```bash
--set serviceAccount.create=false
--set serviceAccount.name=aws-load-balancer-controller
```

Because we already created the service account using eksctl. So Helm should not create a new one.

c)) Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

Expected:

```
READY   UP-TO-DATE   AVAILABLE
2/2     2            2
```

d)) Check pods:

```bash
kubectl get pods -n kube-system | grep aws-load-balancer
```

### Create ACM certificate for ArgoCD domain

1. What is ACM?

ACM means: AWS Certificate Manager. It stores HTTPS certificates.

For our ArgoCD domain: argocd.yourdomain.com, we need an ACM certificate.

#### VVIMP: Important: The ACM certificate must be in the same AWS region as the ALB.

2. Request certificate:

```bash
aws acm request-certificate \
  --domain-name argocd.yourdomain.com \
  --validation-method DNS \
  --region $AWS_REGION
```

This returns: CertificateArn

```json
{
  "CertificateArn": "arn:aws:acm:ap-south-1:123456789012:certificate/abcd..."
}
```

3. Save it:

```bash
export ACM_CERT_ARN="arn:aws:acm:<region>:123456789012:certificate/xxxxxxx"
```

4. Validate certificate with DNS:

ACM will give you a DNS CNAME record. You need to create that record in Route 53. You can get it using:

```bash
aws acm describe-certificate \
  --certificate-arn $ACM_CERT_ARN \
  --region $AWS_REGION \
  --query "Certificate.DomainValidationOptions[0].ResourceRecord"
```

It will show something like:

```json
{
  "Name": "_abc.argocd.yourdomain.com.",
  "Type": "CNAME",
  "Value": "_xyz.acm-validations.aws."
}
```

Wait until: ISSUED ... Do not continue until the certificate is issued.

### Add ACM validation CNAME record

Now you must prove you own the domain. ACM gives you a CNAME record.
You add that CNAME wherever your DNS is managed.

Case 1 — DNS is in Cloudflare
Go to:

```
Cloudflare Dashboard
→ testdomain.com
→ DNS
→ Add record
```

Add:

```
Type: CNAME
Name: _xxxxx.argocd
Target: _yyyyy.acm-validations.aws
Proxy status: DNS only
```

Case 2 — DNS is in GoDaddy
Go to:

```
GoDaddy
→ My Products
→ DNS
→ Add Record
```

Add:

```
Type: CNAME
Host: _xxxxx.argocd
Points to: _yyyyy.acm-validations.aws
```

Case 3 — DNS is in Route 53
If your domain uses Route 53 DNS, you can add the CNAME in Route 53.

#### What is Route 53? How to add CNAME record to Route 53?

Route 53 is AWS DNS service.
Think like this:
Domain registrar = where you bought the domain
DNS provider = where DNS records are controlled

Sometimes both are same.

Example:
Bought domain from GoDaddy
DNS also managed in GoDaddy

But you can also do:
Bought domain from GoDaddy
DNS managed in Route 53
To do that, you create a hosted zone in Route 53, then update the domain’s nameservers at GoDaddy/Cloudflare to the Route 53 nameservers.

### Create ArgoCD Ingress

Now we create the Kubernetes Ingress. This Ingress says:
For argocd.testdomain.com,
create AWS ALB,
use ACM certificate,
route traffic to argocd-server.

Create file: vim argocd-ingress.yaml

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: alb

    alb.ingress.kubernetes.io/scheme: internet-facing

    alb.ingress.kubernetes.io/target-type: ip

    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'

    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:123456789012:certificate/abcd...

    alb.ingress.kubernetes.io/ssl-redirect: "443"

    alb.ingress.kubernetes.io/backend-protocol: HTTP

    alb.ingress.kubernetes.io/healthcheck-path: /healthz

spec:
  ingressClassName: alb
  rules:
    - host: argocd.testdomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 80
```

=> Apply:

```bash
kubectl apply -f argocd-ingress.yaml
```

#### Understand the annotations slowly:

1. kubernetes.io/ingress.class: alb
   Meaning: This Ingress should be handled by AWS Load Balancer Controller.
2. scheme: internet-facing
   Meaning: Create public ALB accessible from the internet.
   Alternative: some companies use:internal ALB-> VPN access only -> private network only
3. target-type: ip
   Meaning: ALB can route directly to pod IPs.
   Alternative: instance -> means ALB routes to worker nodes first.
4. listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
   Meaning: ALB should listen on port 80 and 443. Port 80 will redirect to 443. Port 443 will serve HTTPS.
5. alb.ingress.kubernetes.io/ssl-redirect: "443"
   Meaning: If someone opens http://argocd.testdomain.com, redirect them to https://argocd.testdomain.com.
6. alb.ingress.kubernetes.io/backend-protocol: HTTP
   Meaning: ALB talks to ArgoCD service using HTTP inside the cluster. This is because TLS is already terminated at ALB.
7. alb.ingress.kubernetes.io/healthcheck-path: /healthz
   Meaning: ALB will check /healthz to see if ArgoCD backend is healthy.

#### GitOps version of this setup:

Right now, we applied Ingress manually. But in real GitOps, this file should live in Git. But for the first setup, there is a bootstrapping problem: ArgoCD must exist before ArgoCD can manage things.
So initial installation is often done manually once:
Install ArgoCD->Expose ArgoCD->Connect Git repo->Then ArgoCD manages future changes

#### Helm values version for ArgoCD HTTPS

Instead of creating separate Ingress YAML, you can also configure ArgoCD Helm chart values.
=> Create: vim argocd-values.yaml

```YAML
configs:
  params:
    server.insecure: true

server:
  ingress:
    enabled: true
    ingressClassName: alb
    hosts:
      - argocd.testdomain.com
    paths:
      - /
    pathType: Prefix
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internet-facing
      alb.ingress.kubernetes.io/target-type: ip
      alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
      alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:123456789012:certificate/abcd...
      alb.ingress.kubernetes.io/ssl-redirect: "443"
      alb.ingress.kubernetes.io/backend-protocol: HTTP
      alb.ingress.kubernetes.io/healthcheck-path: /healthz
```

=> Then install:

```bash
helm install argocd argo/argo-cd \
  -n argocd \
  -f argocd-values.yaml
```

OR
Or if already installed:

```bash
helm upgrade argocd argo/argo-cd \
  -n argocd \
  -f argocd-values.yaml
```

### Check if ALB was created

```bash
kubectl get ingress -n argocd
```

You should see:

```
NAME             CLASS   HOSTS                   ADDRESS
argocd-ingress   alb     argocd.testdomain.com   k8s-xxxx.ap-south-1.elb.amazonaws.com
```

The ADDRESS is the ALB DNS name.

### Add DNS record for ArgoCD

Now you need to connect: argocd.testdomain.com to ALB DNS name.
Example ALB DNS: k8s-argocd-xxxx.ap-south-1.elb.amazonaws.com

1. If using Cloudflare:

Go to:

```
Cloudflare
→ DNS
→ Add Record
```

Add:

```
Type: CNAME
Name: argocd
Target: k8s-argocd-xxxx.ap-south-1.elb.amazonaws.com
Proxy status: DNS only
```
