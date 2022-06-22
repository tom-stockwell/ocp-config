# OCP Config

[![CI](https://github.com/stocky37/ocp-config/actions/workflows/ci.yml/badge.svg)](https://github.com/stocky37/ocp-config/actions/workflows/ci.yml)

## Configuring a new cluster
##
```
oc apply -k bootstrap
oc apply -k ../ocp-secrets/bootstrap
oc -n sealed-secrets delete pod -l name=sealed-secrets-controller
```



## App Dependencies
```text
image-registry -> nfs-provisioner
oauth -> sealed-secrets

tenants should always be last
```
