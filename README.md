# OCP Config

[![CI](https://github.com/stocky37/ocp-config/actions/workflows/ci.yml/badge.svg)](https://github.com/stocky37/ocp-config/actions/workflows/ci.yml)

## Configuring a new cluster
##
```sh
cluster=homelab

oc apply -k clusters/$cluster/bootstrap
```
