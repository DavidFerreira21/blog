---
title: "Série k8s: Pods — o básico do Kubernetes"
date: 2026-01-29T11:00:00-03:00
description: "Entenda o que é um Pod no Kubernetes, como criar, inspecionar e aplicar boas práticas com exemplos práticos."
tags: ["k8s", "kubernetes", "pods", "fundamentos", "containers"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/pod.png"
  alt: "Ilustração sobre Pods no Kubernetes"
---

# Série k8s: Pods — o básico do Kubernetes

Este post faz parte da série de fundamentos de Kubernetes. A ideia é apresentar conceitos essenciais em uma sequência prática, do básico ao avançado. Aqui vamos falar de **Pods**, o primeiro bloco de construção de qualquer aplicação no cluster.

Se você está começando, este é o momento de entender o que é um Pod e como ele se comporta. Se já usa Kubernetes, este texto serve como referência rápida com exemplos diretos.

## O que é um Pod

Pod é a menor unidade que o Kubernetes cria e gerencia. Ele funciona como um “envelope” que agrupa um ou mais containers que precisam rodar juntos e compartilhar recursos.

Dentro de um Pod, os containers:

- **Compartilham rede** (mesmo IP e portas);
- **Compartilham volumes** (dados podem ser lidos/escritos entre eles);
- **Compartilham ciclo de vida** (sobem e descem juntos).

Você pode ter dois cenários principais:

- **Pod com um container**: é o caso mais comum. O Pod é apenas o wrapper do container.
- **Pod com múltiplos containers**: usado quando os containers são fortemente acoplados, como um app + sidecar de logs ou proxy.

## Ciclo de vida do Pod

O ciclo de vida de um Pod descreve as fases pelas quais ele passa até ser finalizado. Entender isso ajuda muito no troubleshooting:

- **Pending**: o Pod foi aceito pelo cluster, mas ainda não foi agendado ou as imagens não foram baixadas.
- **Running**: o Pod já foi agendado em um nó e pelo menos um container está em execução.
- **Succeeded**: todos os containers terminaram com sucesso (comum em Jobs).
- **Failed**: algum container terminou com erro e o Pod não conseguiu completar.
- **Unknown**: o estado não pôde ser obtido do nó (problema de comunicação).

Durante a execução, você também verá motivos comuns como `CrashLoopBackOff`, `ImagePullBackOff` e `ErrImagePull`, que indicam problemas de inicialização e download de imagens.

## Init Containers (containers de inicialização)

Init Containers são executados **antes** dos containers principais. Eles são úteis para tarefas de preparação, como:

- baixar configurações;
- verificar dependências;
- preparar volumes ou permissões.

Se um init container falha, o Pod não avança para o estado `Running` até que ele finalize com sucesso.

## Sidecar Containers (containers auxiliares)

Sidecar containers rodam **junto** com o container principal dentro do mesmo Pod. Eles são usados para complementar a aplicação, por exemplo:

- coletar logs;
- fazer proxy/mesh;
- sincronizar arquivos ou configurar cache.

O sidecar compartilha rede e volumes com o container principal, por isso é ideal quando as funções precisam estar bem acopladas.

## Ephemeral Containers (depuração)

Ephemeral containers são containers **temporários** usados para depuração. Eles não fazem parte do manifesto original e podem ser adicionados em um Pod em execução para investigar problemas.

Eles não reiniciam automaticamente e são indicados apenas para troubleshooting.

## Disruptions (interrupções)

Disruptions são eventos que interrompem Pods, como:

- atualização de nós;
- escalonamento do cluster;
- manutenção planejada;
- falhas de infraestrutura.

Para reduzir impacto, usamos **Pod Disruption Budgets (PDBs)**, que definem quantos Pods podem ficar indisponíveis durante uma manutenção.

> Importante: você não usa vários containers no Pod para escalar. Escala se faz criando **mais Pods**, normalmente via Deployment.

Para que os Pods rodem, cada nó precisa ter um **container runtime** instalado (ex.: containerd ou CRI-O).

## Mão na massa

Vamos para alguns comandos e exemplos práticos. Se você ainda não tem um cluster k8s local, utilize o nosso guia de instalação e configuração do Kind: [instalar Kind no WSL](https://davidferreira21.github.io/blog/posts/install-kind/).

## Consultar Pods

- Listar Pods de um namespace específico:

```bash
kubectl get pods -n <namespace>
```

- Ver detalhes do Pod em YAML:

```bash
kubectl get pod <nome-do-pod> -o yaml
```

- Inspecionar eventos e status do Pod:

```bash
kubectl describe pod <nome-do-pod>
```

## Criar e remover Pods

### Criar um Pod simples

Arquivo: `pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-primeiro-pod
  labels:
    run: meu-primeiro-pod
spec:
  containers:
  - name: meu-primeiro-pod
    image: nginx
    ports:
    - containerPort: 80
```

Aplicar o manifesto:

```bash
kubectl apply -f pod.yaml
```

### Criar Pod com múltiplos containers

Arquivo: `pod-mult-container.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-primeiro-pod-multi
  labels:
    run: meu-primeiro-pod-multi
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
  - name: sidecar
    image: alpine
    args:
    - sleep
    - "1800"
```

```bash
kubectl apply -f pod-mult-container.yaml
```

### Remover Pods

- Remover um Pod pelo nome:

```bash
kubectl delete pod <nome-do-pod>
```

- Remover o Pod definido em um arquivo:

```bash
kubectl delete -f pod.yaml
```

## Logs

- Ver logs de um Pod:

```bash
kubectl logs <nome-do-pod>
```

- Acompanhar logs em tempo real:

```bash
kubectl logs -f <nome-do-pod>
```

- Ver logs de um container específico dentro do Pod:

```bash
kubectl logs <nome-do-pod> -c <container-name>
```

## Acesso ao container

O comando `attach` conecta ao processo principal do container em execução:

```bash
kubectl attach <pod-name> -c <container-name>
```

O comando `exec` executa comandos dentro do container:

```bash
kubectl exec <pod-name> -c <container-name> -- <comando>
```

- Abrir shell interativo no container:

```bash
kubectl exec -it ubuntu -- bash
```

## Limites de CPU e memória

Requests definem o mínimo garantido de CPU/memória para o container. Limits definem o máximo que ele pode consumir.

Arquivo: `pod-limits.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-limits
  labels:
    run: pod-com-limits
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "0.5"
      requests:
        memory: "64Mi"
        cpu: "0.3"
```

Aplicar o manifesto:

```bash
kubectl apply -f pod-limits.yaml
```

Validar Pods criados:

```bash
kubectl get pods
```

Opcional: testar consumo com `stress` dentro do container:

```bash
apt update
apt install -y stress
```

```bash
stress --vm 1 --vm-bytes 100M
```

## Volume EmptyDir no Pod

EmptyDir é um volume temporário criado quando o Pod inicia e removido quando o Pod é destruído. É útil para compartilhamento de dados entre containers no mesmo Pod.

Arquivo: `pod-emptydir.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir
spec:
  containers:
  - name: app
    image: ubuntu
    args:
    - sleep
    - infinity
    volumeMounts:
    - name: primeiro-emptydir
      mountPath: /dados
  volumes:
  - name: primeiro-emptydir
    emptyDir:
      sizeLimit: 256Mi
```

Criar Pod com volume EmptyDir:

```bash
kubectl apply -f pod-emptydir.yaml
```

---

No próximo post da série, vamos aprofundar em **Deployments**, com exemplos práticos.
