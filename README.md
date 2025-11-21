# Setup iniziale con FluxCD

Questa guida descrive i passaggi per configurare l’ambiente applicativo `test-apps` su Kubernetes e bootstrap di FluxCD con GitHub.

---

## 1️⃣ Creazione del namespace

Creare il namespace dedicato alle applicazioni:

```bash
kubectl create ns test-apps
```

---

## 2️⃣ Creazione del secret per l’accesso al registry GitHub (GHCR)

Creare il secret `regcred` contenente le credenziali di accesso al GitHub Container Registry.  
Sostituire i valori con quelli della propria organizzazione e con un **Personal Access Token (PAT)** valido.

```bash
kubectl create secret docker-registry regcred   --namespace=test-apps   --docker-server=inserireserverimagine   --docker-username=org-name   --docker-password="INSERIRE_PAT"   --docker-email=emaildpcker
```

> 🔐 **Nota**: assicurarsi che il PAT abbia i permessi per `read:packages` (almeno).

---

## 3️⃣ Bootstrap di FluxCD con GitHub

Avviare il bootstrap di FluxCD puntando al repository GitOps.  
In questo esempio:

- Owner: `nome organizzazione dentro github contenente tutti i progetti `  
- Repository: `nome repository in questo caso test-gitops`  
- Branch: `develop`  
- Path: `clusters/apps "questo path lo crea da solo da non creare"`  

> 🔐 **Nota**: prima di avviare il fluxcd bisognera installare la comandlet con questo 

```bash
winget install fluxcd.flux
```

```bash
flux --version
```

Comando:

```bash
flux bootstrap github   --owner=nomeorganizzazione   --repository=test-gitops   --branch=develop   --path=clusters/apps   --read-write-key=true   --personal   --components-extra=image-reflector-controller,image-automation-controller
```

> 🔐 **Nota**: il --path="cluster/prod" per ogni branch deve essere diverso senno durante il flux andra in conflitto

---

## 4️⃣ Inserimento del PAT

Durante l’esecuzione del comando di bootstrap, verrà richiesto di inserire il **PAT** di GitHub.


## 4️⃣ una volta creato il tutto che crea dentro il repository di github e all'interno del kluster kube utilizzato bisognera collegare i kustomization dentro la cartella 

una volta creato il tutto bisognera aggiungere dentro la cartella clusters/apps e non dentro flux-system che crea in automatico questi collegamenti per apps e i microservizi o applicativi che rilascera ogni 60 secondi dentro kube con pull

da creare con il nome di metadata quindi apps-1microservice.yaml

```bash
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-1microservice
  namespace: flux-system
spec:
  interval: 60s
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./apps/1microservice
  prune: true
  wait: true
```

> 🔐 **Nota**: su name di metadata il nome deve essere apps della cartella e poi il nome della cartella dentro apps e su path su spec mettere il path


## 4️⃣ usare pipeline per rollback

la pipeline si lancia manualmente da actions e bisogna usare lo sha del commit usato e la pipeline va a prendere il commit indietro di 1 e stato progettato per il gitops per le immagini facendo test con il repositori rollback con le immagini cosi che se un immagine non funziona si puo tornare a quella precedente funzionante

## 4️⃣ usare pipeline a livello repository
impostare sui vari repository che si collegano a questo test-gitops la variabile GITOPS_TOKEN A LIVELLO DI REPOSITORY E POI SEMPRE A LIVELLO DI REPOSITORY QUESTO SOTTO

✅ 1. Workflow permissions nel repo
Vai su:
Settings → Actions → General → Workflow permissions

Nel repo che NON funziona, probabilmente è impostato su:

Read-only permissions (default)
Devi cambiarlo in:
Read and write permissions

Questo è il motivo principale per cui il GITHUB_TOKEN non riesce a fare git push.