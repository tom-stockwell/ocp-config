# OCP Config

## Configuring a new cluster

1. Checkout repository
    ```bash
    git clone https://github.com/stocky37/ocp-config.git
    cd ocp-config
    ```
1. Create new cluster directory from an existing cluster
    ```bash
    cluster=<cluster-name>
    cp -r clusters/homelab clusters/$cluster
    ```
1. Update NFS server details in `clusters/$cluster/core/nfs-provisioner/deployment`
1. Update paths for flux & cluster config sync in `clusters/$cluster/flux/flux-system.yml` & `clusters/$cluster/flux/cluster-config.yml` respectively
1. Update htpasswd SealedSecret
    1. Install Sealed Secrets
        ```bash
        kustomize build bases/core/sealed-secrets | oc apply -f -
        ```
        > :exclamation: Once OpenShift is updated to Kubernetes 1.21  `oc apply -k bases/core/sealed-secrets` can be used.
    1. Generate new htpasswd SealedSecret
    1. Replace `clusters/$cluster/core/auth/secret-htpasswd.yml` with the new SealedSecret
1. Commit & push
    ```bash
    git add .
    git commit -m "Add $cluster cluster"
    git push
    ```
1. Apply the cluster flux config
    ```bash
    kustomize build clusters/$cluster/flux | oc apply -f -
    ```