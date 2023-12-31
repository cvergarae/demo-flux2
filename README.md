# demo de flux2 

## Pre-requisitos

## Instalación Flux en MacOS

**Instalación en MacOS**
```
brew install fluxcd/tap/flux
```
**Check flux**
```
flux check --pre
```

## Instalación Flux en Kubernetes

### Bootstraping en Github

**Credenciales Github**
```
export GITHUB_TOKEN=<PAT-token>
```

**Bootstrap en github**
```
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --token-auth \
  --owner=cvergarae \
  --repository=demo-flux2 \
  --branch=main \
  --path=clusters/demo-cluster \
  --read-write-key \
  --personal
```

## Despliegue Aplicación podinfo

## ver estado de despliegue
```
flux get kustomizations --watch
```

# Creación Política de inagen

**Creación repositorio de imágenes**
```
flux create image repository podinfo \
--image=layer0/podinfo \
--interval=5m \
--export > ./clusters/demo-cluster/podinfo-registry.yaml
```

**Creación política de atualización**

```
flux create image policy podinfo \
--image-ref=podinfo \
--filter-regex='^dev-(?P<ts>.*)' \
--filter-extract='$ts' \
--select-numeric='asc' \
--export > ./clusters/demo-cluster/podinfo-policy.yaml
```

**Creación configuración de actualización de imágenes**

```
flux create image update flux-system \
--interval=30m \
--git-repo-ref=flux-system \
--git-repo-path="./clusters/demo-cluster" \
--checkout-branch=main \
--push-branch=main \
--author-name=fluxcdbot \
--author-email=fluxcdbot@users.noreply.github.com \
--commit-template="{{range .Updated.Images}}{{println .}}{{end}}" \
--export > ./clusters/demo-cluster/flux-system-automation.yaml
```



```
---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: podinfo
  namespace: flux-system
spec:
  filterTags:
    pattern: '^dev-(?P<ts>.*)'
    extract: '$ts'
  policy:
    numerical:
      order: asc
  imageRepositoryRef:
    name: podinfo
```

# Instalación GitOps Weaveworks

```
brew tap weaveworks/tap
brew install weaveworks/tap/gitops
```
```
PASSWORD="<A new password you create, removing the brackets and including the quotation marks>"
```
```
gitops create dashboard ww-gitops \
  --password=$PASSWORD \
  --export > ./clusters/demo-cluster/weave-gitops-dashboard.yaml
```

```
kubectl port-forward svc/ww-gitops-weave-gitops -n flux-system 9001:9001
```


# Comandos varios


```
docker build . -t layer0/podinfo:dev-8
```

```
docker push layer0/podinfo:dev-8
```

```
flux reconcile kustomization --with-source flux-system
```