# Secrets mit SOPS und Helm Secrets für ArgoCD

Hier wird die Verwendung von SOPS und Helm Secrets für ArgoCD beschrieben. Hierfür werden seperate values.yaml Dateien für die Secrets verwendet. Diese werden dann mit dem Helm Plugin `helm secrets` verschlüsselt und in einem Git Repository abgelegt. ArgoCD kann dann die verschlüsselten Secrets entschlüsseln und an die Applikationen weitergeben. Dieses Repository enthält ein Beispiel für die Verwendung von SOPS und Helm Secrets für ArgoCD.

## Voraussetzungen

- [helm secrets](https://github.com/jkroepke/helm-secrets)
- Public Key

Damit `helm secrets` verwendet werden kann, muss das Plugin installiert werden. Dazu kann folgender Befehl verwendet werden:

```bash
helm plugin install https://github.com/jkroepke/helm-secrets
```

Zudem muss die Konfigurationsdatei `.sops.yaml` im Projektverzeichnis vorhanden sein. Diese Datei enthält den Public Key für die Verschlüsselung der Secrets. Ein Beispiel für die Konfigurationsdatei ist im Repository vorhanden.

## Verwendung
In diesem Beispiel werden die Secrets in einer seperaten values.yaml Datei ausgelagert. Diese Datei wird dann mit `helm secrets` verschlüsselt. Die Ordnerstruktur sieht wie folgt aus:

```
.
├── README.md
├── templates
│   └── ...
├── secrets
│   ├── dev
│   │   ├── values.yaml     # unverschlüsselte Secrets (nicht im Repository)
│   │   └── values.enc.yaml # verschlüsselte Secrets (im Repository)
│   ├── stage
│   │   ├── values.yaml     # unverschlüsselte Secrets (nicht im Repository)
│   │   └── values.enc.yaml # verschlüsselte Secrets (im Repository)
│   └── prod
│       ├── values.yaml     # unverschlüsselte Secrets (nicht im Repository)
│       └── values.enc.yaml # verschlüsselte Secrets (im Repository)
├── values
│   ├── dev.yaml    # values.yaml für dev
│   ├── stage.yaml  # values.yaml für stage
│   └── prod.yaml   # values.yaml für prod
├── .sops.yaml
└── values.yaml     # globale values.yaml
```

Wird nun ein Secret beispielsweise für die `dev` Umgebung hinzugefügt, so wird die `values.yaml` Datei im `secrets/dev` Verzeichnis angelegt. Diese Datei wird dann mit `helm secrets` verschlüsselt und in `values.enc.yaml` gespeichert. Die verschlüsselte Datei kann dann im Git Repository abgelegt. 

```bash
helm secrets encrypt secrets/prod/values.yaml > secrets/prod/values.enc.yaml
```

In ArgoCD muss dann der Pfad zur verschlüsselten Datei mit angegeben werden. ArgoCD kann die Datei dann entschlüsseln und an die Applikation weitergeben.

## Verschlüsseltes Secret bearbeiten

Um ein verschlüsseltes Secret zu bearbeiten, kann folgender Befehl verwendet werden. Dazu wird allerdings der Private Key benötigt, welche zuerst importiert werden muss. Um den Private Key zu importieren, muss dieser unter dem Pfad `$XDG_CONFIG_HOME/sops/age/keys.txt` hinterlegt werden. Nun kann eine verschlüsselte Datei mit `helm secrets` bearbeitet werden.

```bash
helm secrets edit secrets/dev/values.enc.yaml
```

Es öffnet sich ein Editor, in dem die Datei bearbeitet werden kann. Nach dem Speichern wird die Datei wieder verschlüsselt.

## Helm Chart deployen ohne ArgoCD

Mit `helm secrets` können die verschlüsselten Secrets auch ohne ArgoCD deployt werden. Dazu wird folgender Befehl verwendet:

```bash
helm secrets upgrade --install my-release . -f secrets/dev/values.enc.yaml -f values/dev.yaml
```
