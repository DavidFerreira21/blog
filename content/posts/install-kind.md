---
title: "Guia rápido: Kind no WSL com Docker Desktop"
date: 2026-01-28
tags: ["k8s", "kubernetes", "kind", "wsl2", "docker-desktop", "windows"]
categories: ["k8s", "devops"]
draft: false
cover:
  image: "images/kind-wsl-cover.svg"
  alt: "Arte abstrata para post sobre Kind no WSL"
---

# Guia rápido: Kind no WSL com Docker Desktop

Este guia mostra como preparar o WSL2, integrar o Docker Desktop e subir um cluster Kind local para testes e labs.

## Pré-requisitos (Windows)
### 1) Instalar/ativar o WSL e definir o Ubuntu como padrão
Abra o PowerShell como Administrador e execute:
```powershell
wsl --install
wsl --set-default-version 2
wsl --install -d Ubuntu
```
Defina o Ubuntu como distro padrão:
```powershell
wsl --set-default Ubuntu
```
Reinicie o Windows se for solicitado.

### 2) Instalar o Docker Desktop
1. Baixe e instale o Docker Desktop para Windows.
2. Abra o Docker Desktop e finalize o onboarding.

### 3) Habilitar WSL Integration no Docker Desktop
No Docker Desktop:
1. **Settings** → **Resources** → **WSL Integration**
2. Marque **Enable integration with my default WSL distro**
3. Ative a integração para **Ubuntu**
4. Clique em **Apply & Restart**

### 4) Validar WSL + Docker dentro do Ubuntu
Abra o Ubuntu (WSL) e rode:
```bash
wsl --list --verbose
docker version
docker ps
```
Siga adiante somente se o Docker estiver respondendo sem erros dentro do Ubuntu.

## Instalar o Kind
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/kind
```

## Instalar o kubectl (se ainda nao tiver)
```bash
sudo apt-get update
sudo apt-get install -y kubectl
```

## Validar Docker + Kind
```bash
docker ps        # verifica se o Docker Engine está rodando
kind version     # confirma se o Kind está acessível
```
Siga apenas se ambos os comandos executarem sem erros.

## Criar o cluster Kind
1. Crie um arquivo `kind-config.yaml` (exemplo abaixo) e ajuste se necessario:
   ```yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
     - role: control-plane
     - role: worker
     - role: worker
   ```
2. Suba o cluster:
   ```bash
   kind create cluster --name dev --config kind-config.yaml
   ```
3. Valide o contexto `kind-dev`:
   ```bash
   kubectl cluster-info --context kind-dev
   kubectl get nodes
   ```

## Instalar o Ingress NGINX
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## Comandos úteis
```bash
kind get clusters   # lista clusters
kubectl get nodes   # status dos nodes
kind delete cluster --name dev
```
Utilize `make down` ao final dos testes para liberar recursos.

## Dicas rápidas
- Se o Docker nao responder no WSL, reinicie o Docker Desktop.
- Garanta que a integracao WSL esteja ativa para a distro Ubuntu.
- Evite nomes de cluster duplicados se estiver testando mais de um.
