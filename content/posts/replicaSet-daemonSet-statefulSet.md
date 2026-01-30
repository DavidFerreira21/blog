---
title: "Série k8s: ReplicaSet, DaemonSet e StatefulSet — quando usar cada um"
date: 2026-01-29T14:30:00-03:00
description: "Entenda as diferenças entre ReplicaSet, DaemonSet e StatefulSet, com casos de uso e comandos práticos."
tags: ["k8s", "kubernetes", "replicaset", "daemonset", "statefulset", "fundamentos"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/workloads.png"
  alt: "Ilustração sobre workloads no Kubernetes"
---

# Série k8s: ReplicaSet, DaemonSet e StatefulSet — quando usar cada um

Este post faz parte da série de fundamentos de Kubernetes. Aqui vamos entender **três tipos de workloads** que aparecem o tempo todo no dia a dia: ReplicaSet, DaemonSet e StatefulSet.

A ideia é explicar o papel de cada um, quando usar e quais comandos básicos ajudam na operação.


## ReplicaSet

O ReplicaSet garante que uma quantidade desejada de Pods esteja sempre em execução. Na prática, ele é quase sempre **criado e gerenciado por um Deployment**.

Use o ReplicaSet quando você quer manter um número fixo de Pods iguais rodando. O controlador cria Pods quando faltam e remove quando sobram.

### Exemplo YAML (ReplicaSet)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
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

### Comandos (ReplicaSet)

- Aplicar um Deployment (gera o ReplicaSet definido no manifesto):

```bash
kubectl apply -f deployment.yaml
```

Exemplo `deployment.yaml`:

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

- Listar ReplicaSets:

```bash
kubectl get rs
```

- Detalhar um ReplicaSet (status, eventos e pods associados):

```bash
kubectl describe rs
```

- Remover um ReplicaSet pelo nome:

```bash
kubectl delete replicaset nginx-replicaset
```

- Remover o ReplicaSet definido em um arquivo:

```bash
kubectl delete -f nginx-replicaset.yaml
```

## DaemonSet

O DaemonSet garante que **todos os nós do cluster** (ou um subconjunto) executem uma cópia de um Pod. É ideal para agentes que precisam rodar em cada nó.

Casos comuns:

- agentes de monitoramento (ex.: node exporter);
- coletores de logs (ex.: Fluent Bit/Fluentd);
- componentes de rede (ex.: CNI, kube-proxy).

### Exemplo YAML (DaemonSet)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
```

### Comandos (DaemonSet)

- Aplicar o manifesto do DaemonSet:

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

- Listar DaemonSets:

```bash
kubectl get daemonset
```

- Listar os Pods do DaemonSet pelo label:

```bash
kubectl get pods -l app=node-exporter
```

- Listar os nós do cluster (para conferir onde o DaemonSet vai rodar):

```bash
kubectl get nodes
```

- Remover o DaemonSet pelo nome:

```bash
kubectl delete daemonset node-exporter
```

- Remover o DaemonSet definido em um arquivo:

```bash
kubectl delete -f node-exporter-daemonset.yaml
```

## StatefulSet

O StatefulSet é o controlador recomendado para **aplicações com estado**. Ele garante:

- identidade estável dos Pods (nome previsível);
- ordem de criação e remoção;
- volumes persistentes exclusivos por réplica.

É fundamental para bancos de dados e sistemas distribuídos (PostgreSQL, MongoDB, Kafka, Zookeeper, etc.).

Além disso, StatefulSet normalmente depende de um **Headless Service** para garantir identidades de rede estáveis.

### Exemplo YAML (StatefulSet)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-stateful
  labels:
    app: nginx-stateful
spec:
  serviceName: nginx-headless
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: [\"ReadWriteOnce\"]
        resources:
          requests:
            storage: 1Gi
```

### Comandos (StatefulSet)

- Aplicar o manifesto do StatefulSet (Service headless + StatefulSet):

```bash
kubectl apply -f statefulset.yaml
```

- Listar StatefulSets:

```bash
kubectl get statefulset
```

- Detalhar um StatefulSet:

```bash
kubectl describe statefulset nginx-stateful
```

- Listar os Pods do StatefulSet:

```bash
kubectl get pods -l app=nginx-stateful
```

- Listar PVCs criados pelo StatefulSet:

```bash
kubectl get pvc
```

- Remover o StatefulSet pelo nome:

```bash
kubectl delete statefulset nginx-stateful
```

- Remover o StatefulSet definido em um arquivo:

```bash
kubectl delete -f statefulset.yaml
```

---

No próximo post da série, seguimos detalhando **Services** e como expor aplicações dentro e fora do cluster.
