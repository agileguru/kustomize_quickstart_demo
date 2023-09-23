# Kustomize Quick Start Demo

A repository for a ready to use as template for [Kustomize][def3]  

## Assumptions 

1. You have a public DNS for your website
1. Using nignx ingress controller as described in [here][def2]
1. Using kcert SSL manager for your public site as described [here][def2]
1. Have a working kubectl and respective kube config configuration. If you dont have the configuraation then you can execute the following command 

    > gcloud container clusters get-credentials &lt;cluster name&gt; --zone &lt;cluster zone&gt; --project &lt;project id having the cluster&gt;

## Install kubectl CLI. 

Follow the documentation at [https://kubernetes.io/docs/tasks/tools/][def]

## Creating the folder structure ( best practice )

You need to create 2 folders 

1. Base : this is the folder to create the manifest templates with almost the correct values. One folder for each component is a good idea. Something like below... 

```
- base
    - api-1
        - api-1.yaml
        - kustomize.yaml
    - api-2 
        - api-1.yaml
        - kustomize.yaml
    - ingress 
        - apps.yaml
        - kustomize.yaml    
```    

1. Overlays : this is the folder to create various "envionment" specific folders having metadata for each environment. Something like below... 

```
- overlays
    - dev
        - namespace.yaml
        - api-1-override.yaml
        - api-2-override.yaml
        - sit-ingress-patch.json
        - kustomize.yaml
    - sit
        - namespace.yaml
        - api-1-override.yaml
        - api-2-override.yaml
        - sit-ingress-patch.json
        - kustomize.yaml
```

## Customize each environment using kustomize.yaml  

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: dev-overlay

namespace: dev

resources:
  - namespace.yaml
  - ../../base/configs/dev
  - ../../base/api-1
  - ../../base/api-2
  - ../../base/ingress


patchesStrategicMerge:
- api-1-override.yaml
- api-2-override.yaml  

patches:
- target:
    kind: Ingress
    name: app-ingress
  path: dev-ingress-patch.json
 ```

### How to Work with kustomize 

1. Change the host in the ingress and remove the comment from dev-ingress-patch.json and sit-ingress-patch.json else you will get error like
 
```
error: trouble configuring builtin PatchTransformer with config: `
path: dev-ingress-patch.json
target:
  kind: Ingress
  name: app-ingress
: unable to parse SM or JSON patch from [[
    {
        "op": "replace",
        "path": "/spec/rules/0/host",
        "value": "dev.agileguru.org" // Change the host here and remove this comment
    },
    {
        "op": "replace",
        "path": "/spec/tls/0/hosts/0",
        "value": "dev.agileguru.org" // Change the host here and remove this comment
    }
]]
```
2. Do a dry-run and then apply using commands given below using  

```
$  kubectl apply -k overlays/dev --dry-run=client
$  kubectl apply -k overlays/sit --dry-run=client
```
### Things To You Need To Do
1. Keep the Replica Count to 0 in the base configuration
1. Always Specify the Namespace in Overlay kustomization.yaml
1. Always Dry Run with yaml output to ensure accuracy
1. Good Naming Conventions for folders and manifest files
1. Keep Ingress Mapping in its own-folder
1. Always have override.yaml in for each component. 

### Things You Should Not Do
1. Hard-Coding namespace in Base Configurations
1. Mix Configurations and Application Code In the Same Folder
1. Git branches for Environment configuration ( know as  parity drift )  

[def]: https://kubernetes.io/docs/tasks/tools/
[def2]: https://github.com/agileguru/gke_nginx_kcert_quick_start
[def3]: https://www.kustomize.io