# SOPS mit age für ArgoCD mit Kustomize und Helm - Showcase

Dieses Repository dient als Showcase für die Verwendung von [SOPS](https://github.com/getsops/sops) zur Ver- und Entschlüsselung von Kubernetes Secrets in ArgoCD. Als Verschlüsselungsalgorithmus wird [age](https://github.com/FiloSottile/age) verwendet. Mit SOPS können Secrets in Kubernetes Secrets verschlüsselt werden, um sensible Daten sicher zu speichern. Hier wird gezeigt, wie SOPS in Verbindung mit ArgoCD, Kustomize und Helm verwendet werden kann.

## Beschreibung
SOPS ist ein Tool, mit dem Dateien ver- und entschlüsselt werden können. Dabei wird ein Secret in einer YAML-Datei verschlüsselt und in einem Kubernetes-Secret gespeichert. SOPS unterstützt verschiedene Verschlüsselungsalgorithmen wie age, PGP oder KMS. In diesem Showcase wird age als Verschlüsselungsalgorithmus verwendet. SOPS kann in Verbindung mit ArgoCD verwendet werden, um Secrets in Kubernetes Secrets zu ver- und entschlüsseln. Dazu wird SOPS als Plugin in ArgoCD eingesetzt. Außerdem wird gezeigt, wie SOPS in Verbindung mit Kustomize und Helm verwendet werden kann. Dabei wird für Kustomize das Plugin [`KSOPS`](https://github.com/viaduct-ai/kustomize-sops) und für Helm das Plugin [`helm-secrets`](https://github.com/jkroepke/helm-secrets) verwendet. Beide Plugins ermöglichen die Entschlüsselung von Secrets in Kustomize und Helm mittels SOPS in ArgoCD.

## Voraussetzungen

- [age](https://github.com/FiloSottile/age)
- [sops](https://github.com/getsops/sops)
- [kustomize](https://github.com/kubernetes-sigs/kustomize) (wenn Kustomize verwendet wird)
- [helm](https://helm.sh/) (wenn Helm verwendet wird)

## Age Key erstellen und im Cluster hinterlegen

Für die Verwendung von age muss ein Key erstellt werden. Dieser Key wird verwendet, um die Secrets zu verschlüsseln und zu entschlüsseln. Der Key wird in einem Secret `sops-age` im Kubernetes Cluster hinterlegt.

```bash
age-keygen -o age.agekey
cat age.agekey | kubectl create secret generic sops-age --namespace argocd --from-file=keys.txt=/dev/stdin
```

## ArgoCD Konfiguration

Um SOPS mit ArgoCD zu verwenden, muss die ArgoCD Konfiguration angepasst werden. Dazu stehen in diesem Repository zwei YAML-Dateien zur Verfügung (`argocd-cm-patch.yaml` und `argocd-deployment-patch.yaml`). Diese Dateien enthalten patches für ArgoCD, um SOPS zu verwenden. Im Kubernetes Cluster müssen nun diese Patches angewendet werden.

```bash
kubectl patch configmap argocd-cm -n argocd --patch "$(cat argocd-cm-patch.yaml)"
kubectl patch deployment argocd-repo-server -n argocd --patch "$(cat argocd-deployment-patch.yaml)"
```

Oder ohne dieses Repository klonen zu müssen:

```bash
kubectl patch configmap argocd-cm -n argocd --patch "$(curl https://raw.githubusercontent.com/choffmann/sops-showcase/main/argocd-cm-patch.yaml)"
kubectl patch deployment argocd-repo-server -n argocd --patch "$(curl https://raw.githubusercontent.com/choffmann/sops-showcase/main/argocd-deployment-patch.yaml)"
```

## Wie verschlüssel ich ein Secret?
Die verschiedene Anwendungsbeispiele für Kustomize und Helm werden im jeweiligen Unterordner [`helm`](https://github.com/choffmann/sops-showcase/tree/main/helm) oder [`kustomize`](https://github.com/choffmann/sops-showcase/tree/main/kustomize) beschrieben.


