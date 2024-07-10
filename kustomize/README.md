# Secrets in Kubernetes mit SOPS und age verschlüsseln mit Kustomize und ArgoCD

Wenn ArgoCD erfolgreich konfiguriert wurde, kann über SOPS Secrets verschlüsselt werden. Dazu wird das Plugin [SOPS](https://github.com/getsops/sops) benötigt. In Kustomize wird zudem ein Secrets-Generator erstellt, der die Secrets aus einer verschlüsselten Datei entschlüsselt. Dieser Prozess wird in ArgoCD stattfinden, um die Secrets in Kubernetes Secrets zu überführen.

## Voraussetzungen

- [SOPS](https://github.com/getsops/sops)
- Public Key 

Für die Verschlüsselung von Secrets wird der Public Key benötigt, welcher im Cluster hinterlegt wurde. Dieser Key wird in der `.sops.yaml` Datei unter dem Schlüssel `age` hinterlegt. Diese Datei wird benötigt, um SOPS zu konfigurieren. Der Schlüssel `unencrypted_regex` definiert, welche Werte nicht verschlüsselt werden sollen.

```yaml
creation_rules:
  - unencrypted_regex: "^(apiVersion|metadata|kind|type)$"
    age: <age-public-key>
```

## Secrets verschlüsseln

Um nun ein Secrets zu verschlüsseln, wird die `sops` CLI verwendet.

```bash
sops -e secret.yaml > secret.enc.yaml
```

`secret.yaml` ist die unverschlüsselte Secret Datei aus Kubernetes und `secret.enc.yaml` ist das verschlüsselte Secret. Diese verschlüsselte Dateikann auch in einem Git Repository abgelegt werden. Wenn ArgoCD auf das Repository zugreift, wird das Secret automatisch entschlüsselt und in Kubernetes Secrets überführt.

**NOTE:** Die Secrets werden trotzdem noch in `base64` angegeben, da hier lediglich die Secrets Datei von Kubernets verschlüsselt wird. 

## Kustomize Generator

Damit ein Secret in ArgoCD entschlüsselt werden kann, muss in Kustomize ein Generator erstellt werden.

```yaml
apiVersion: viaduct.ai/v1
kind: ksops
metadata:
  # Specify a name
  name: example-secret-generator
  annotations:
    config.kubernetes.io/function: |
      exec:
        # if the binary is in your PATH, you can do
        path: ksops
        # otherwise, path should be relative to manifest files, like
        # path: /path/to/ksops
files:
  - ./secret.enc.yaml
```

Dieser Generator wird in der `kustomization.yaml` Datei referenziert.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
generators:
  - ./secret-generator.yaml
```
