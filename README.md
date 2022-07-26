# OpenShift Cluster Configuration

[![CI](https://github.com/tom-stockwell/ocp-config/actions/workflows/ci.yml/badge.svg)](https://github.com/tom-stockwell/ocp-config/actions/workflows/ci.yml)

This repository configures different OpenShift clusters that I use regularly.

## Layout

* [clusters](#clusters)
    * &lt;cluster&gt;
        * bootstrap 
        * &lt;various-components&gt;
* [infra](#infrastructure)

This repository uses [Kustomize](https://kustomize.io/) to configure all the necessary Kubernetes resources.
However, I use the `.yml` extensions for Kustomization files rather than the standard `.yaml` extension as a personal preference.

The [clusters](#clusters) directory contains configuration for each cluster.
See [Bootstrapping a cluster](#bootstrapping-a-cluster) to get started with a new cluster.
Once bootstrapped, an Argo CD Application will be automatically created for each directory (except for `bootstrap`) under each cluster directory using an ApplicationSet.

Therefore, for each component a cluster will want to use, a subdirectory is created with a `kustomization.yml` file.
This Kustomization should use the equivalent `component` directory as a base before implementing any cluster-specific configuration.

The base cluster subdirectory contains a Kustomization file which includes all the individual components in order to allow the configuration of the cluster as a whole to be tested in CI.

The [infra](#infrastructure) directory contains various pieces of configuration.
These are typically sourced from external, shared repositories and could be omitted here and sourced remotely in the cluster configurations.
However, I have still included them here in case there are any personal modifications I would like to make that could be shared amongst all my clusters.

## Bootstrapping a cluster

1. Install the openshift-gitops & sealed-secrets operators. The process has been simplified with the following command.
    ```sh
    cluster=homelab
    oc apply -k clusters/$cluster/bootstrap/0-operators
    ```
2. Once the operators are installed, configure the ArgoCD instance and, optionally, the shared sealed secrets key.
    ```
    oc apply -k clusters/$cluster/bootstrap/1-gitops-instance
    ```
    ```sh
    namespace=sealed-secrets
    label=sealedsecrets.bitnami.com/sealed-secrets-key

    # deactivate existing active secrets
    oc get secrets -n "$namespace" -l "$label=active" -o name | xargs -I{} oc label -n "$namespace" {} "$label-"
       
    # add shared secrets as the active cert
    # NOTE: ensure they are labelled as the active cert
    oc apply -f <secret>
    
    # delete controller to pick up new secret
    oc -n "$namespace" delete pod -l name=sealed-secrets-controller
    ```
   > **NOTE:** If you opt not to use a shared sealed secrets key you will need to regenerate all sealed secrets for the cluster in question.
3. Once Argo CD & Sealed Secrets are configured, create the ArgoCD applications.
    ```
    oc apply -k clusters/$cluster/bootstrap/2-gitops
    ```
    Argo CD will now configure the rest of the cluster, and manage itself and its applications. 

## Clusters

| Cluster                                                         | Description                                                                               |
|-----------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| **[homelab](./clusters/homelab)**                               | Configuration for my homelab cluster                                                      |
| **[rhpds-demo-gitops-cicd](./clusters/rhpds-demo-gitops-cicd)** | Configuration for setting up the GitOps & CI/CD demo on an RHPDS Open Environment cluster | 

## Infrastructure

| Component                                                               | Description                                                                                                                                                                                                  |
|-------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **[acm](infra/acm)**                                             | Installs & configures [Red Hat Advanced Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) (**Note:** out of date)                                           |
| **[gitops](infra/gitops)**                                       | Configures the ArgoCD Applications for cluster configuration                                                                                                                                                 |
| **[image-registry](infra/image-registry)**                       | Configures the [Image Registry Operator](https://docs.openshift.com/container-platform/4.10/registry/configuring-registry-operator.html)                                                                     |
| **[namespace-config](infra/namespace-config)**                   | Installs the [Namespace Configuration Operator](https://github.com/redhat-cop/namespace-configuration-operator)                                                                                              |
| **[nfs-provisioner](infra/nfs-provisioner)**                     | Installs the [NFS Subdir External Provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)                                                                                           |
| **[oauth](infra/oauth)**                                         | Configures [OpenShift OAuth identity providers](https://docs.openshift.com/container-platform/4.10/authentication/understanding-identity-provider.html)                                                      |
| **[openshift-gitops-instance](infra/openshift-gitops-instance)** | Configures the cluster-wide ArgoCD instance                                                                                                                                                                  |
| **[openshift-gitops-operator](infra/openshift-gitops-operator)** | Installs [OpenShift GitOps](https://docs.openshift.com/container-platform/4.10/cicd/gitops/understanding-openshift-gitops.html)                                                                              |
| **[openshift-pipelines](infra/openshift-pipelines)**             | Installs [OpenShift Pipelines](https://docs.openshift.com/container-platform/4.10/cicd/pipelines/understanding-openshift-pipelines.html)                                                                     |
| **[sealed-secrets](infra/sealed-secrets)**                       | Installs [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)                                                                                                                                    |                                                                                                                                                                   
| **[user-workload-monitoring](infra/user-workload-monitoring)**   | Enables [User Workload Monitoring](https://docs.openshift.com/container-platform/4.10/monitoring/enabling-monitoring-for-user-defined-projects.html) in the cluster monitoring stack (**Note:** out of date) |
