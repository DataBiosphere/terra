# Personal Test Environments
Personallly provisioned test environments, and the steps to create and manage them, are described here.
## Background
### Use Case
Personal Kubernetes environments allow realistic testing of deployed services without interfering
with shared environments. This enables faster development cycles, as code can be tested without
waiting for a release, and without risk of breaking colleagues' environments.

### Where they Live
A personal environment is contained by a K8s [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
named for a developer, sharing a larger cluster in the GCP
project [terra-kernel-k8s](https://console.cloud.google.com/home/dashboard?project=terra-kernel-k8s)
called 
[terra-integration](https://console.cloud.google.com/kubernetes/clusters/details/us-central1-a/terra-integration?project=terra-kernel-k8s&tab=details&persistent_volumes_tablesize=50&storage_class_tablesize=50&nodes_tablesize=50&node_pool_tablesize=10).
Kubernetes [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) target this namespace and allow `kubectl` commands to be narrowly focused. The deployment
consists of one or more
[pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [service](https://kubernetes.io/docs/concepts/services-networking/service/)
in the case of Workspace Manager, though the system is more flexible than that. A user may wish to include
several services in a personal environment, depending on their testing needs. A given Terra service
deployment could involve more than one k8s pod or deployment. The term deployment is also used to mean
a deployment of the entire Terra application.

## Prerequisites
THe basic prerequisites are
1. An `@broadinstitute.org` account
2. Membership in the Google group [dsde-engineering](https://groups.google.com/a/broadinstitute.org/g/dsde-engineering)
for access to private Github repos and [ArgoCD](https://argoproj.github.io/argo-cd/)
3. A public GitHub account
4. A linkage between this GitHub account and a Broad account, using the [Broad Institute GitHub App](https://broad-github.appspot.com/). 
5. (helpful) SSH credentials for using GitHub
6. Membership in the private Git team for Platform Foundations.
7. A first initial and last name. Using `jcarlton` in these examples.

For most permissions issues, contact DevOps.

See also the [DSP's Getting Started with `kubectl`](https://docs.dsp-devops.broadinstitute.org/kubernetes/kubectl-getting-started)
and [Terraform Best Practices](https://docs.dsp-devops.broadinstitute.org/best-practices-guides/terraform).  
## Step 1: Terraform Orchestration for Terra Environment
Terra uses terraform to set up cloud services, accounts, and artifacts that our services need, but not
for deploying the services themselves. These resources are all managed outside of Kubernetes.
At this point, the [existing documentation](https://docs.dsp-devops.broadinstitute.org/mc-terra/mcterra-quickstarts#Create-a-new-namespaced-environment-in-an-existing-cluster)
in the DevOps Confluence is applicable, and some of the following borrows from it. The process involves
two different pull requests in different repositories. The first is a change to the 
[terraform-ap-depoyments](https://github.com/broadinstitute/terraform-ap-deployments)
repo, and the subject is any cloud dependencies that must be instantiated for the new environment. Different
use cases will have different needs here, but for Workspace Manager all that is needed is the following.

The Broad's Terraform modules live in a public repository called [terraform-ap-modules](https://github.com/broadinstitute/terraform-ap-modules).
These are combined with generated private client modules, variables, and secrets to instantiate the
actual environments and artifacts. The latter are contained in the private repo [terraform-ap-deployments](https://github.com/broadinstitute/terraform-ap-deployments), 
and this is the one to change. See this [DevOps Portal entry](https://docs.dsp-devops.broadinstitute.org/mc-terra/mcterra-deployment#Infrastructure---Terraform)
for more background.

First, declare a new Terra Environment in [atlantis.yaml](https://github.com/broadinstitute/terraform-ap-deployments/blob/master/atlantis.yaml#L70). A new object
in a yaml array describes the top-level properties of the environment. Note that "environment" here
is a set of conventions for creating a stand-alone instance of one or more services, and not anything
specific in k8s or Terraform. Mine looks like this:
```yaml
- name: terra-env-jcarlton
  dir: terra-env
  workflow: terra-env-jcarlton
  workspace: jcarlton
  autoplan:
    enabled: false
```
It's easy to get confused between the env name (`terra-env-jcarlton`), the workspace (`jcarlton`), and the k8s namespace eventually
created (`terra-jcarlton`). There are numerous examples of the conventions in this file, and following
existing examples works well. Note that Atlantis is a tool for managing Terraform updates,
but atlantis.yaml is not itself a Terraform file.

Second, specify some workflow settings for deploying the environment. Among other things, this tells
Atlantis to pass the variables file `jcarlton.tfvars`
```yaml
  terra-env-jcarlton:
    plan:
      steps:
        - init:
            extra_args: ["-upgrade"]
        - plan:
            extra_args: ["-var-file", "tfvars/jcarlton.tfvars"]
```

Third, check in a tfvars file for Terraform to use as its input variables when building this [workspace](https://www.terraform.io/docs/state/workspaces.html).
THe contents of these files change frequently and are service-dependent.

This file should be in a path like `terraform-ap-deployments/terra-env/tfvars/jcarlton.tfvars`. 
Folder IDs refer to [GCP Project Folders](https://cloud.google.com/resource-manager/docs/creating-managing-folders).
The entries under `terra_apps` describe services a deployment will depend on. A `true` value causes
resources for a dependency to be created. Google project and cluster information should be the same
for everyone.

The repository is integrated with [Atlantis](https://www.runatlantis.io/), which checks terraform artifacts
before merge to make sure they can `plan` and `apply` successfully. When pushing a PR to this repo, 
one must run `atlantis` commands as part of the review process. A detailed protocol for
PRs in this repo is in its [README file](https://github.com/broadinstitute/terraform-ap-deployments/blob/master/README.md).
First, a github comment of
 ```shell script
 atlantis plan -p terra-env-jcarlton
```
to which `broadbot` will reply:
```text
Ran Plan for project: terra-env-jcarlton dir: terra-env workspace: jcarlton

Show Output
```

Following a successful `plan` and a review from one of the CODEOWNERS  (ping `#dsp-devops-champions`
for a reviewer), wait for an approval. Then run `atlantis apply -p terra-env-jcarlton`, and if that's successful,
merge the PR.
## Step 2: Edit the Helm File and Auxiliary Files
[Helm](https://helm.sh/) manages k8s packages called [Helm Charts](https://helm.sh/docs/topics/charts/),
allowing large-scale Kubernetes deployments. These may be published in public or private repositories,
similarly to Docker container applications. In turn, a [helmfile](https://github.com/roboll/helmfile)
may be used to work with many charts in a large, heterogeneous application (like Terra).

### Aside: look through the Charts
The main Terra Helm charts don't need changing for a personal environment, but set up instructions to create
an instance with it. Charts for Terra are located in the public [terra-helm repo](https://github.com/broadinstitute/terra-helm).
The chart directory for workspace manager is [here](https://github.com/broadinstitute/terra-helm/tree/master/charts/workspacemanager).
THe chart is actually surprisingly brief, showing only one dependency.
```yaml
apiVersion: v2
name: workspacemanager
version: 0.19.0
description: Chart for Terra Workspace Manager
type: application
keywords:
  - dsp
  - broadinstitute
  - terra
  - workspacemanager
sources:
  - https://github.com/broadinstitute/terra-helm/tree/master/charts
  - https://github.com/DataBiosphere/terra-workspace-manager
dependencies:
  - name: postgres
    condition: postgres.enabled
    version: 0.1.0
    repository: https://broadinstitute.jfrog.io/broadinstitute/terra-helm-proxy
```

It's also possible to navigate the helm repo itself using the `helm` command:
```shell script
$ helm repo add terra-helm https://broadinstitute.github.io/terra-helm
```
```shell script
$ helm repo update
```
```text
...Successfully got an update from the "terra-helm" chart repository
```

Then just do 
```shell script
$ helm pull terra-helm/workspacemanager
```
and a .tgz file such as `workspacemanager-0.19.0.tgz` should be downloaded silently to the current directory.

No changes to the `terra-helm` repo are necessary in order to set up a personal environment.

### The Helmfile
The helmfile allows specifying override values for a chart for a particular
deployment, and the [terra-helmfile repo](https://github.com/broadinstitute/terra-helmfile)
is the place providing and for editing those. This "gitOps" model
is similar to the relationship between the private, Atlantis-enabled `terraform-ap-deployments` repo
and the public `terraform-ap-modules` that described above.

It's not necessary to have `helm` installed locally for this step, though it will be required later.


Clone the [terra-helmfile](https://github.com/broadinstitute/terra-helmfile) and look at the file structure.
```shell script
$ tree -L 2
```

```
.
├── README.md
├── argocd
│   └── values
├── bin
│   ├── logging.sh
│   └── render
├── clusters.yaml
├── docs
│   └── NAMING.md
├── envDefaults.yaml
├── environments
│   ├── live
│   ├── live.yaml
│   ├── personal
│   ├── personal.yaml
│   └── preview.yaml
├── helmfile.yaml
├── terra
│   └── values
├── versions
│   ├── README.md
│   ├── alpha.yaml
│   ├── prod.yaml
│   └── staging.yaml
└── versions.yaml

```
There is a main `helmfile.yaml`, an `environments` directory with subdirectories for `live` and `personal`, and a `terra` directory
with subdirectories for each service:
```shell script
jaycarlton~/repos/terra-helmfile/terra (master) $ tree -L 2
```

```
.
└── values
    ├── agora
    ├── buffer
    ├── buffer.yaml
    ├── consent
    ├── crljanitor
    ├── crljanitor.yaml
    ├── cromiam
    ├── cromwell
    ├── datarepo
    ├── datarepomonitoring
    ├── duos
    ├── firecloudorch
    ├── global
    ├── global.yaml
    ├── hydra
    ├── icdemo
    ├── identityconcentrator
    ├── istioconfig
    ├── istioconfig.yaml
    ├── leonardo
    ├── ontology
    ├── poc
    ├── poc.yaml
    ├── rawls
    ├── sam
    ├── workspacemanager
    └── workspacemanager.yaml
```
For this PR we'll edit all of
* the helmfile itself, to make it aware of our personal enviornment
* a personal.yaml file defining our environment's enabled services at a  high level.
* several values files defining parameters for the individual services in the environment

### Changes to helmfile.yaml
The main helmfile itself gets a simple two-line entry for this project pointing to other files with
properties to read (including the one above):
```yaml
  jcarlton:
    values:
      - environments/personal.yaml
      - environments/personal/jcarlton.yaml
```

### Changes to Personal yaml file
This file describes the Terra apps that are required for the personal environment. For simplicity,
one cna look at an environment with only workspace manager enabled. For the file `terra-helmfile/environments/personal/jcarlton.yaml`:
```yaml
# Environment overrides for jcarlton environment
releases:
  workspacemanager:
    enabled: true
```
If any applications are disabled, their resources are not generated by helm.

The main personal.yaml is nearly empty (and should not be edited). It appears just to be a lightweight
bit of plumbing so the whole repo isn't just one massive file.
<details>

<summary>personal.yaml</summary>

```yaml
# Common settings for private developer environments
base: 'personal'

# All private dev environments go to same cluster
defaultCluster: terra-integration
```

</details>

Finally, each service to be included has its own `values` override file for our environment:
For `terra/values/workspacemanager/personal/jcarlton.yaml`, verify that [vault](https://www.vaultproject.io/) has a secret
stored at the path described. In my case this had already been taken care of.

```yaml
vault:
  pathPrefix: secret/dsde/terra/kernel/integration/jcarlton

ingress:
  staticIpName: terra-integration-jcarlton-workspace-ip
```

## Step 3: Creating the namespace
### `kubectl` Setup
Interaction with Kubernetes is most often accomplished with `kubectl`. This is a local utility that is packaged
with Kubernetes. Type `kubectl version` to make sure it's installed.
```shell script
$ kubectl version
```
<details>

```text
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"17+", GitVersion:"v1.17.14-gke.400", GitCommit:"b00b302d1576bfe28ae50661585cc516fda2227e", GitTreeState:"clean", BuildDate:"2020-11-19T09:22:49Z", GoVersion:"go1.13.15b4", Compiler:"gc", Platform:"linux/amd64"}
```

</details>

 To connect `kubectl` to the cluster, first update `kubeconfig` with the proper credentials. 
`gcloud container` has a [clusters get-credentials](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
option to support exactly this use case. In the present case case it is necessary to specify both the
project and zone as well as the cluster name:  
```shell script
$ gcloud container clusters get-credentials terra-integration --zone us-central1-a --project terra-kernel-k8s
```
```text
Fetching cluster endpoint and auth data.
kubeconfig entry generated for terra-integration.
```

### Making the namespace
```shell script
$ kubectl create namespace terra-jcarlton
```
```text
namespace/terra-jcarlton created
```

##  Deploying (WorkspaceManager-specific)
The Workspace Manager repository has a [dev-local directory](https://github.com/DataBiosphere/terra-workspace-manager/tree/dev/local-dev)
with instructions in its README for loading the helm chart and terraform state onto a pod in the new namespace. 

## Syncing in ArgoCD
The [terra-app-generator](https://ap-argocd.dsp-devops.broadinstitute.org/applications/terra-app-generator)
application in ArgoCD needs to sync the new environment and its dependent sources. As of this writing,
only a member of DevOps engineers and admins have access to this app. So contact someone in #dsp-devops-champions and ask to have
the new app synced.
## Using `kubectl` 
To view the current `kubectl config` do
```shell script
$ kubectl config view
```

<details>

<summary>Config view output</summary>

```text
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://34.70.205.77
  name: gke_terra-kernel-k8s_us-central1-a_terra-integration
contexts:
- context:
    cluster: gke_terra-kernel-k8s_us-central1-a_terra-integration
    namespace: terra-jcarlton
    user: gke_terra-kernel-k8s_us-central1-a_terra-integration
  name: gke_terra-kernel-k8s_us-central1-a_terra-integration
current-context: gke_terra-kernel-k8s_us-central1-a_terra-integration
kind: Config
preferences: {}
users:
- name: gke_broad-dsde-dev_us-central1-a_terra-dev
  user:
    auth-provider:
      config:
        cmd-args: config config-helper --format=json
        cmd-path: /Users/jaycarlton/google-cloud-sdk/bin/gcloud
        expiry-key: '{.credential.token_expiry}'
        token-key: '{.credential.access_token}'
      name: gcp
- name: gke_terra-kernel-k8s_us-central1-a_terra-integration
  user:
    auth-provider:
      config:
        access-token: <<REDACTED>>
        cmd-args: config config-helper --format=json
        cmd-path: /Users/jaycarlton/google-cloud-sdk/bin/gcloud
        expiry: "2021-01-21T19:54:04Z"
        expiry-key: '{.credential.token_expiry}'
        token-key: '{.credential.access_token}'
      name: gcp
```
</details>

The credentials and other info are contained in a `kubectl context` and can be inspected:
```shell script
kubectl config get-contexts
```
```text
CURRENT   NAME                                                   CLUSTER                                                AUTHINFO                                               NAMESPACE
*         gke_terra-kernel-k8s_us-central1-a_terra-integration   gke_terra-kernel-k8s_us-central1-a_terra-integration   gke_terra-kernel-k8s_us-central1-a_terra-integration   terra-jcarlton
```
### Getting Descriptions
To get the description of the namespace, use `describe namespace`
```shell script
$ kubectl describe namespace terra-jcarlton
```
```text
Name:         terra-jcarlton
Labels:       <none>
Annotations:  <none>
Status:       Active

Resource Quotas
 Name:                       gke-resource-quotas
 Resource                    Used  Hard
 --------                    ---   ---
 count/ingresses.extensions  1     5k
 count/jobs.batch            0     10k
 pods                        1     5k
 services                    1     1500

No LimitRange resource.
```

The output shows an active namespace with no labels and plenty of quota. To view details
about the pods, use `get pods` and isolate the namespace to use:
```shell script
$ kubectl get pods --namespace=terra-jcarlton
```
```text
NAME                                           READY   STATUS     RESTARTS   AGE
workspacemanager-deployment-68dfc86fbb-zhpwc   0/3     Init:0/1   0          4h35m
```
 For the service, use `get services`
 ```shell script
$ kubectl get services --namespace=terra-jcarlton
```
```text
NAME                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
workspacemanager-service   ClusterIP   10.56.8.103   <none>        443/TCP   4h39m
```

It's also possible to set this namespace as a default, and omit the `--namespace`
qualifier:
```shell script
$ kubectl config set-context --current --namespace=terra-jcarlton
```
```text
Context "gke_terra-kernel-k8s_us-central1-a_terra-integration" modified.
```

To list the nodes in the cluster do
```shell script
kubectl get nodes
```

<details>

<summary>Node list</summary>

```text
NAME                                             STATUS   ROLES    AGE   VERSION
gke-terra-integration-cronjob-v1-9d4ae6cb-dvz6   Ready    <none>   31h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-hv62      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-jrmn      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-m2ui      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-pa8n      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-qstb      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-c1ae3b79-rert      Ready    <none>   30h   v1.17.14-gke.400
gke-terra-integration-default-v2-95d71f95-gdfy   Ready    <none>   30h   v1.17.14-gke.400
```

</details>

### Running Resiliency Tests Through Test Runner Library
Test Runner Library provides the underlying framework for running resiliency tests within namespaces. The whole process involves only a few simple setup and configuration steps described below.

At a high level, running resiliency tests within namespaces require a set of permissions to manipulate cluster resources with Kubernetes API.
These permissions are namespace scoped so that no resiliency tests will have cluster-wide access.

The required namespace permissions are specified in the 3 manifest templates which comes with the Test Runner Library distribution.
The `setup-k8s-testrunner.sh` script templates the formation of the actual manifests for deploying to a namespace.
The `setup-k8s-testrunner.sh` script also carries out the following functions:

* Provision the Kubernetes Service Account, RBAC Role and RoleBinding for Test Runner.
* Export credentials of the Test Runner Kubernetes Service Account to Vault.

To set up a namespace for Test Runner resiliency tests, simply run the command in the following example (`terra-zloery` namespace for example).

The first argument is the `kubectl context` mentioned elsewhere in this document.

The second argument is the Terra namespace (without the `terra-` prefix).

The third argument is just some text to describe the application itself.
```shell script
$ ./setup-k8s-testrunner.sh gke_terra-kernel-k8s_us-central1-a_terra-integration zloery workspacemanager
```

Once the script above ran successfully, the namespace is ready for resiliency testing through the Test Runner Framework.

The Kubernetes credentials stored in Vault needs to be rendered by means of the `./render-k8s-config.sh zloery` script in the test repository before kicking off the resiliency tests.
```shell script
$ ./render-k8s-config.sh zloery
```

<summary>testrunner-k8s-serviceaccount.yml.template</summary>

```text
# Do not modify this template file.

# This template file is used for setting a Test Runner K8S Service Account
# for running resiliency tests in a namespace.
#
# This template file is to be used in conjunction with the other template files
#
#   testrunner-k8s-role.yml.template
#   testrunner-k8s-rolebinding.yml.template
#
# within an automation pipeline and is not meant to be run separately or manually.

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: APP
  name: testrunner-k8s-sa
  namespace: terra-NAMESPACE
```

<summary>testrunner-k8s-role.yml.template</summary>

```text
# Do not modify this template file.

# This template file is used for setting a Test Runner privileged RBAC role
# for running resiliency tests in a namespace.
#
# This template file is to be used in conjunction with the other template files
#
#   testrunner-k8s-sa.yml.template
#   testrunner-k8s-rolebinding.yml.template
#
# within an automation pipeline and is not meant to be run separately or manually.

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: testrunner-k8s-role
  # A k8s namespace: e.g. terra-wsmtest, terra-ichang.
  # Avoid using default or system namespaces such as kube-system.
  namespace: terra-NAMESPACE
  labels:
    app.kubernetes.io/component: APP
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/exec"]
    verbs: ["get", "list", "watch", "delete", "patch", "create", "update"]
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments", "deployments/scale"]
    verbs: ["get", "list", "watch", "delete", "patch", "create", "update"]
```
<summary>testrunner-k8s-rolebinding.yml.template</summary>

```text
# Do not modify this template file.

# This template file is used for binding a Test Runner K8S Service Account
# to a privileged RBAC role for running resiliency tests in a namespace.
#
# This template file is to be used in conjunction with the other template files
#
#   testrunner-k8s-sa.yml.template
#   testrunner-k8s-role.yml.template
#
# within an automation pipeline and is not meant to be run separately or manually.

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: testrunner-k8s-sa-rolebinding
  # A k8s namespace: e.g. terra-wsmtest, terra-ichang.
  # Avoid using default or system namespaces such as kube-system.
  namespace: terra-NAMESPACE
  labels:
    app.kubernetes.io/component: APP
subjects:
  # Authorize In-Cluster Service Account
  - kind: ServiceAccount
    name: testrunner-k8s-sa
    namespace: terra-NAMESPACE
roleRef:
  kind: Role
  name: testrunner-k8s-role
  apiGroup: rbac.authorization.k8s.io
```

### Additional Background Reading
#### Kubernetes
[The Kubernetes Book]() is very accessible for beginners (like the author), and uses lots of images and
repetition to bring the main points home. Additionally, the exercises on the main Kubernetes site are
well-paced, insightful, and a realistic facsimile of a k8s cluster and the process of interacting
with it. The [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) has lots
of useful goodies.

#### Terraform
[Terraform Up & Running](https://www.amazon.com/Terraform-Running-Writing-Infrastructure-Code/dp/1491977086)
is an excellent resource for getting one's head around Terraform and some of the
design decisions that went into it.
