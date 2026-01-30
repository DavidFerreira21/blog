---
title: "Série k8s: Deployments — como versionar e escalar aplicações"
date: 2026-01-29T13:00:00-03:00
description: "Entenda o que é um Deployment no Kubernetes, como funciona rollout/rollback e as principais estratégias de atualização."
tags: ["k8s", "kubernetes", "deployments", "rollout", "fundamentos"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/deployment.png"
  alt: "Ilustração sobre Deployments no Kubernetes"
---

# Série k8s: Deployments — como versionar e escalar aplicações

Este post faz parte da série de fundamentos de Kubernetes. Aqui vamos falar de **Deployments**, o recurso que mantém aplicações rodando com réplicas, atualizações seguras e possibilidade de rollback.

Se você já entendeu Pods, agora o Deployment é o próximo passo natural para rodar aplicações de forma confiável.

## O que você vai ver aqui

- O que é um Deployment e quando usar
- Um exemplo YAML pronto para aplicar
- Estratégias de rollout e comandos de operação

## O que é um Deployment

Um Deployment é o recurso mais usado para rodar aplicações **stateless** no Kubernetes. Ele gerencia o ciclo de vida de Pods por meio de **ReplicaSets** e garante que a aplicação esteja sempre no estado desejado.

Na prática, um Deployment permite:

- criar e manter **réplicas**;
- fazer **rollout** (atualização gradual);
- fazer **rollback** (voltar para uma versão anterior);
- pausar e retomar mudanças com controle.

Se um Pod falha, o Deployment cria outro automaticamente. Se você altera a imagem, ele troca os Pods aos poucos, mantendo a aplicação disponível.

## Quando usar

Deployments são a escolha padrão para workloads **stateless** (APIs, front-ends, workers). Para workloads **stateful**, o recurso mais adequado geralmente é o **StatefulSet**.

Se você precisa atualizar versões sem downtime e manter réplicas estáveis, o Deployment é o caminho mais seguro.

## Exemplo de Deployment (YAML)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

## Aplicar e verificar

- Aplicar o Deployment definido em `deployment.yaml`:

```bash
kubectl apply -f deployment.yaml
```

- Verificar se o Deployment foi criado e está saudável:

```bash
kubectl get deployments -l app=nginx
```

- Listar os Pods criados pelo Deployment:

```bash
kubectl get pods -l app=nginx
```

- Detalhar o Deployment (status, eventos e estratégia):

```bash
kubectl describe deployment nginx-deployment
```

- Listar os ReplicaSets gerados pelo Deployment:

```bash
kubectl get replicasets -l app=nginx
```

## Estratégias de rollout

O Kubernetes possui duas estratégias principais para atualizar um Deployment:

- **RollingUpdate (padrão)**: substitui Pods aos poucos, mantendo parte das réplicas antigas ativas para evitar indisponibilidade. Ideal para produção.
- **Recreate**: encerra todas as réplicas antigas e depois cria as novas, causando indisponibilidade temporária. Útil quando você não pode ter versões diferentes rodando juntas.

Você também pode configurar parâmetros do rolling update, como `maxUnavailable` e `maxSurge`, para definir o ritmo da atualização.

Exemplo: permitir no máximo 1 indisponível e criar 1 extra durante o rollout.

## Rollout (atualizações)

- Acompanhar o status do rollout (geral):

```bash
kubectl rollout status
```

- Acompanhar o status do rollout de um Deployment específico:

```bash
kubectl rollout status deployment nginx-deployment
```

- Ver o histórico de revisões do Deployment (geral):

```bash
kubectl rollout history
```

- Ver o histórico detalhado de uma revisão:

```bash
kubectl rollout history deployment nginx-deployment --revision=1
```

- Voltar para a versão anterior do Deployment (geral):

```bash
kubectl rollout undo
```

- Voltar para a versão anterior de um Deployment específico:

```bash
kubectl rollout undo deployment nginx-deployment
```

- Pausar temporariamente o rollout do Deployment:

```bash
kubectl rollout pause
```

- Retomar o rollout do Deployment após a pausa:

```bash
kubectl rollout resume
```

- Reiniciar o rollout do Deployment (força nova atualização sem alterar a imagem):

```bash
kubectl rollout restart
```

- Reiniciar o rollout de um Deployment específico:

```bash
kubectl rollout restart deployment nginx-deployment
```

## Remover o Deployment

- Remover o Deployment pelo nome:

```bash
kubectl delete deployment nginx-deployment
```

- Remover o Deployment definido em um arquivo:

```bash
kubectl delete -f deployment.yaml
```

---

No próximo post da série, vamos detalhar **Services** e como expor aplicações dentro e fora do cluster.
