---
title: "Série k8s: Probes — liveness, readiness e startup"
date: 2026-01-29T15:30:00-03:00
description: "Entenda quando usar liveness, readiness e startup probes, com exemplos YAML práticos."
tags: ["k8s", "kubernetes", "probes", "observabilidade", "fundamentos"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/probes.png"
  alt: "Ilustração sobre probes no Kubernetes"
---

# Série k8s: Probes — liveness, readiness e startup

Este post faz parte da série de fundamentos de Kubernetes. Aqui vamos entender **probes**, as verificações de saúde que o Kubernetes usa para decidir se um container deve reiniciar, entrar no balanceamento ou aguardar inicialização.

## O que são probes

Probes são checagens de saúde executadas pelo kubelet **dentro do container**. Elas são configuradas por container e podem usar três mecanismos:

- `httpGet`: testa um endpoint HTTP.
- `tcpSocket`: testa abertura de porta TCP.
- `exec`: executa um comando dentro do container.

## Tipos de probes e quando usar

- **livenessProbe**: responde à pergunta **“esse container ainda está vivo?”**.  
  Se falhar repetidamente, o kubelet **reinicia o container**. É ideal para detectar deadlocks, travamentos ou situações em que o processo “parece vivo”, mas não responde mais.

- **readinessProbe**: responde **“esse container está pronto para receber tráfego?”**.  
  Se falhar, o Pod **é removido do balanceamento** (Service endpoints), mas o container não é reiniciado. É útil quando a aplicação precisa de tempo para aquecer, carregar cache ou reconectar dependências.

- **startupProbe**: responde **“a aplicação já subiu?”**.  
  É usada quando o container demora a inicializar. Enquanto a startupProbe não passar, o Kubernetes **ignora liveness e readiness**, evitando reinícios prematuros durante o boot.

## Liveness Probe

Verifica se o processo está saudável. Se falhar repetidamente, o Pod é reiniciado.

Exemplo completo (`nginx-liveness.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-liveness
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      livenessProbe:
        tcpSocket:
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 5
        failureThreshold: 3
```

## Readiness Probe

Controla quando o Pod entra no balanceamento de carga do Service. Se falhar, o Pod fica indisponível para tráfego, mas não reinicia.

Exemplo completo (`nginx-readiness.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-readiness
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 5
        successThreshold: 2
        failureThreshold: 3
```

## Startup Probe

Usada para containers que demoram a subir. Enquanto ela não passar, liveness/readiness ficam desativadas.

Exemplo completo (`nginx-startup.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-startup
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      startupProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 5
        successThreshold: 1
        failureThreshold: 30
```

## Comandos rápidos

- Aplicar o manifesto da liveness probe:

```bash
kubectl apply -f nginx-liveness.yaml
```

- Aplicar o manifesto da readiness probe:

```bash
kubectl apply -f nginx-readiness.yaml
```

- Aplicar o manifesto da startup probe:

```bash
kubectl apply -f nginx-startup.yaml
```

- Verificar pods e status das probes:

```bash
kubectl get pods
```

- Ver detalhes e eventos do pod:

```bash
kubectl describe pod <nome-do-pod>
```

---

No próximo post da série, seguimos com **Services** e como expor aplicações no cluster.
