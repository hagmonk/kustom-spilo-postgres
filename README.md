# kustom-spilo-postgres

[Kustomize](https://github.com/kubernetes-sigs/kustomize)-enabled manifests for
deploying [zalando/spilo](https://github.com/zalando/spilo) HA PostgreSQL
clusters into Kubernetes.

## Usage

Create a `kustomization.yaml` file in your own repo. Here's an example:

``` yaml
bases:
  - git::ssh://git@github.com/hagmonk/kustom-spilo-postgres
  
commonLabels:
  cluster-name: mini
```

Ensure that environment variables `SUPERUSER_PASSWORD` and
`REPLICATION_PASSWORD` are set when running the `kustomize build` command:

``` sh
env SUPERUSER_PASSWORD=hovercraft REPLICATION_PASSWORD=eels kustomize build
```

This **should** product valid YAML that can be piped directly to `kubectl apply
-f -`

## Configuration

[Kustomize](https://github.com/kubernetes-sigs/kustomize) has excellent
documentation and examples, but here are some ideas to get you started for
overriding config.

The underlying Spilo image is configured through environment variables, which
are documented here for
[Patroni](https://patroni.readthedocs.io/en/latest/ENVIRONMENT.html) and here
for [Spilo](https://github.com/zalando/spilo/blob/master/ENVIRONMENT.rst).

`kustomization.yaml`

``` yaml
bases:
  - git::ssh://git@github.com/hagmonk/kustom-spilo-postgres

commonLabels:
  cluster-name: mini

# Target a different namespace
namespace: wonderland 

# Prefix all resource names with this
namePrefix: production-

# Merge new environment variables in
configMapGenerator:
- name: postgres
  behavior: merge
  literals: 
    - PGUSER_ADMIN=lord-foobar

# Tweak the statefulset for replicas and storage
patches:
  - statefulset-patch.yaml
```

`statefulset-patch.yaml`

``` yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: postgres
spec:
  replicas: 3
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        storageClassName: hyper-ssd-gold-edition
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 300Gi

```

