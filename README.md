## Tigera Private Installation

Questa repository contiene istruzioni su come deployare il [Tigera Operator](https://github.com/tigera/operator) per Calico usando un container registry privato.

1. Installare [Skopeo](https://github.com/containers/skopeo).
2. Accedere con Skopeo (`skopeo login`) al registry privato. Se si sta usando Quay, utilizzare le credenziali alla voce "Docker Login" generabili in *Account Settings > Docker CLI and Applications*.

3. Trasferire le immagini dal proprio computer locale al registry. Supponendo che il registry privato sia `quay.dev`, eseguire i seguenti comandi:

```bash
skopeo copy docker://quay.io/tigera/operator:v1.34.3 docker://quay.dev/tigera/operator:v1.34.3
skopeo copy docker://calico/apiserver:v3.28.1 docker://quay.dev/calico/apiserver:v3.28.1
skopeo copy docker://calico/typha:v3.28.1 docker://quay.dev/calico/typha:v3.28.1
skopeo copy docker://calico/ctl:v3.28.1 docker://quay.dev/calico/ctl:v3.28.1
skopeo copy docker://calico/node:v3.28.1 docker://quay.dev/calico/node:v3.28.1
skopeo copy docker://calico/cni:v3.28.1 docker://quay.dev/calico/cni:v3.28.1
skopeo copy docker://calico/kube-controllers:v3.28.1 docker://quay.dev/calico/kube-controllers:v3.28.1
skopeo copy docker://calico/dikastes:v3.28.1 docker://quay.dev/calico/dikastes:v3.28.1
skopeo copy docker://calico/pod2daemon-flexvol:v3.28.1 docker://quay.dev/calico/pod2daemon-flexvol:v3.28.1
skopeo copy docker://calico/csi:v3.28.1 docker://quay.dev/calico/csi:v3.28.1
skopeo copy docker://calico/node-driver-registrar:v3.28.1 docker://quay.dev/calico/node-driver-registrar:v3.28.1
```

*Nota*: è ovviamente possibile alterare i tag delle immagini in base alle proprie esigenze.

4. Dallo stesso pannello da cui sono state prelevate le credenziali per il login con Skopeo, dovrebbe essere possibile anche produrre un oggetto Secret per Kubernetes da iniettare nei pod per permettere il pull delle immagini. Curarsi, dunque, di produrre questo oggetto Secret, e di chiamarlo (nei metadati) `registry-pull-secret`. Inserirlo dunque al path `manifests/registry_secret.yml`.

Il file ha una forma di questo tipo:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-pull-secret
data:
  .dockerconfigjson: eW...=
type: kubernetes.io/dockerconfigjson
```

5. In `manifests/tigera_operator.yml`, configurare questo spezzone di codice nell'unico Deployment presente nel file sostituendo a `quay.dev` il proprio registry:

```yaml
      containers:
        - name: tigera-operator
          image: quay.dev/tigera/operator:v1.34.3
```

Analogamente, configurare in maniera opportuna il file `manifests/tigera_configuration.yml`.

```yaml
spec:
  # Configures Calico networking.
  imagePullSecrets:
    - name: registry-pull-secret
  registry: "quay.dev"
```

6. Eseguire i seguenti comandi.

```bash
kubectl create -f manifests/registry_secret.yml -n tigera-operator
kubectl create -f manifests/tigera_operator.yaml

# Appena l'Operator è up-and-running...
kubectl create -f manifests/registry_secret.yml
kubectl create -f manifests/tigera_configuration.yaml
```

## Riferimenti

- [Tigera Private Registry Configuration](https://docs.tigera.io/calico/latest/operations/image-options/alternate-registry)
- [Tigera Installation Reference](https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Image)