# eximiait-cluster-config-demo

## Pre-requisitos del sistema

1. Docker
2. [KinD](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

## Configuración el cluster demo

El resultado de ejecutar los siguientes pasos es un cluster de Kubernetes (KinD) con los siguientes componentes:

- ArgoCD.
- Sealed-Secrets (solamente Secret con certificado para encriptar/desencriptar los secrets).

El resto de las herramientas y configuraciones del cluster se instalarán a través de GitOps utilizando ArgoCD.

### Pasos

1. Crear cluster con Kind (omitir si ya se dispone de un cluster de K8S/KinD):

    ```bash
    kind create cluster --config kind/config.yaml
    ```

1. Desencriptar el Secret para Sealed-Secrets:

    1. Crear el archivo `ansible-vault-password` con la contraseña provista con el comando a continuación. Luego, este archivo se usará para desencriptar el archivo `./helmsman/secrets/kind-cluster-tls.yaml.vault`:

        ```bash
        echo "<REDACTED>" > ansible-vault-password
        ```

    1. Ejecutar el siguiente comando para desencriptar el secret necesario para la herramienta Sealed-Secrets:

        ```bash
        docker run --network host --rm -it -v ${PWD}:/workdir --name ansible eximiait/cluster-config-tools ansible-vault decrypt /workdir/helmsman/secrets/kind-cluster-tls.yaml.vault --out /workdir/charts/sealed-secret-config/templates/kind-cluster-tls.yaml --vault-password-file /workdir/ansible-vault-password
        ```

1. Configuración Cluster:

    1. Ejectuar el contenedor:

        ```bash
        docker run --network host --rm -it -v ${PWD}:/workdir -v ${HOME}/.kube:/root/.kube --name my-container eximiait/cluster-config-tools
        ```

    1. Revisar configuración de Helmsman en `helmsman/helmsman.yaml`, por ejemplo que el valor de *kubeContext* sea el correcto.
    1. Ejecutar Helmsman:

        ```bash
        root@docker-demo: helmsman -f helmsman/helmsman.yaml --apply --subst-env-values
        ```

## Argocd

- Para obtener la password del usuario admin:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
