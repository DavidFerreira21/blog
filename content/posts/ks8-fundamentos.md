---
title: "Série k8s: Fundamentos de Kubernetes"
date: 2026-01-29T09:00:00-03:00
description: "Início da série de Kubernetes: conceitos base, runtimes, componentes e visão geral da arquitetura."
tags: ["k8s", "kubernetes", "fundamentos", "containers", "cloud"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/k8s-fundamentos-cover.png"
  alt: "Ilustração sobre fundamentos de Kubernetes"
---

# Fundamentos de Kubernetes (k8s) — Parte 1

Esta é a primeira parte de uma série de posts sobre Kubernetes. A ideia é construir uma base sólida, do básico ao avançado, passando por arquitetura, workloads, rede, storage, segurança, observabilidade e boas práticas de operação.

Se você está começando agora, este post é o ponto de partida. Se já usa Kubernetes no dia a dia, use como revisão rápida dos conceitos fundamentais.

## Sobre esta série

Nesta série vamos abordar alguns fundamentos:

- Conceitos base de containers e runtimes
- Componentes do Kubernetes e arquitetura do cluster
- Workloads (pods, deployments, jobs)
- Rede e exposição de serviços
- Storage e persistência
- Segurança e boas práticas operacionais
- Observabilidade e troubleshooting

## Para quem é este conteúdo

- Quem está começando em Kubernetes e quer uma base sólida
- Quem já usa k8s, mas precisa organizar os conceitos
- Quem quer material de referência para o dia a dia


## O que é o Kubernetes?

O projeto Kubernetes foi desenvolvido pela Google, em meados de 2014, para atuar como um orquestrador de contêineres para a empresa. O Kubernetes (k8s), cujo termo em grego significa "timoneiro", é um projeto *open source* que conta com *design* e desenvolvimento baseados no projeto Borg, que também é da Google. Alguns outros produtos disponíveis no mercado, tais como o Apache Mesos e o Cloud Foundry, também surgiram a partir do projeto Borg.

Como Kubernetes é uma palavra difícil de se pronunciar e escrever, a comunidade simplesmente o apelidou de **k8s**, seguindo o padrão i18n (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".


### Alguns sites que devemos visitar

Abaixo temos os sites oficiais do projeto do Kubernetes:

- https://kubernetes.io
- https://github.com/kubernetes/kubernetes/
- https://github.com/kubernetes/kubernetes/issues

Abaixo temos as páginas oficiais das certificações do Kubernetes (CKA, CKAD e CKS):

- https://www.cncf.io/certification/cka/
- https://www.cncf.io/certification/ckad/
- https://www.cncf.io/certification/cks/

## O container engine

Antes de começar a falar um pouco mais sobre o Kubernetes, precisamos entender alguns componentes importantes no ecossistema. Um desses componentes é o container engine.

O *container engine* é o responsável por gerenciar imagens e volumes, garantindo o isolamento dos recursos que os containers estão utilizando (CPU, memória, storage, rede, etc.).

Hoje temos diversas opções para se utilizar como *container engine*, que até pouco tempo atrás eram quase sinônimo de Docker.

Opções como Docker, CRI-O e Podman são bem conhecidas e preparadas para ambiente produtivo. O Docker, como todos sabem, é o container engine mais popular e utiliza o containerd como runtime.

**Container runtime?** O que é isso?

Calma que vou explicar já já, mas antes temos que falar sobre a OCI. :)

### OCI — Open Container Initiative

A OCI é uma organização sem fins lucrativos que tem como objetivo padronizar a criação de containers, para que possam ser executados em qualquer ambiente. A OCI foi fundada em 2015 pela Docker, CoreOS, Google, IBM, Microsoft, Red Hat e VMware e hoje faz parte da Linux Foundation.

O principal projeto criado pela OCI é o *runc*, que é o principal container runtime de baixo nível, e utilizado por diferentes *container engines*, como o Docker.

O *runc* é um projeto open source, escrito em Go, e seu código está disponível no GitHub.

Agora sim já podemos falar sobre o que é o container runtime.

### O container runtime

Para que seja possível executar containers nos nós, é necessário ter um *container runtime* instalado em cada um deles.

O *container runtime* é o responsável por executar os containers nos nós. Quando você está utilizando Docker ou Podman para executar containers na sua máquina, você está fazendo uso de algum *container runtime* (ou melhor, o seu container engine está fazendo uso de algum runtime).

Temos quatro tipos de *container runtime*:

- **Low-level**: executados diretamente pelo Kernel, como runc, crun e runsc.
- **High-level**: executados por um *container engine*, como containerd, CRI-O e Podman.
- **Sandbox**: executados por um *container engine* e responsáveis por executar containers de forma segura, usando uma camada adicional. O gVisor é um exemplo desse tipo.
- **Virtualized**: executados por um *container engine* e responsáveis por executar containers de forma segura em máquinas virtuais. A performance é menor do que quando são executados nativamente. O Kata Containers é um exemplo.


## Arquitetura do k8s

Assim como os demais orquestradores disponíveis, o k8s segue um modelo *control plane/workers*, constituindo assim um *cluster*, onde para seu funcionamento é recomendado no mínimo três nós: o nó *control plane*, responsável (por padrão) pelo gerenciamento do *cluster*, e os demais como *workers*, executores das aplicações que queremos executar sobre esse *cluster*.

É possível criar um cluster Kubernetes rodando em apenas um nó, porém é recomendado somente para fins de estudos e nunca para produção.

Caso você queira utilizar o Kubernetes em sua máquina local, existem diversas soluções que criam um cluster Kubernetes usando VMs ou Docker. Alguns exemplos:

- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start): cluster Kubernetes com containers Docker. Útil para estudos, desenvolvimento e testes. **Não usar em produção**.
- [Minikube](https://github.com/kubernetes/minikube): cluster local com um nó. **Não usar em produção**.
- [MicroK8s](https://microk8s.io): pode ser usado em produção, especialmente para Edge e IoT.
- [k3s](https://k3s.io): distribuição leve, executa inclusive em Raspberry Pi.
- [k0s](https://k0sproject.io): distribuição em binário único, focada em simplicidade. **Pode ser usada em produção**.

### Componentes principais

- **API Server**: fornece a API (JSON/HTTP) do cluster. A comunicação com o cluster ocorre principalmente via `kubectl`.
- **etcd**: *datastore* chave-valor distribuído que armazena o estado do cluster. Por padrão roda no control plane.
- **Scheduler**: escolhe o nó onde um *pod* será executado, baseado em recursos e políticas.
- **Controller Manager**: garante que o estado atual do cluster converge para o estado desejado (ex.: número de réplicas).
- **Kubelet**: é o agente do k8s que roda nos nós workers. Ele gerencia os *pods* em cada nó.
- **Kube-proxy**: responsável pelo roteamento e pelo balanceamento de tráfego dentro do nó.

## Portas que devemos nos preocupar

**Control plane**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|6443*|Kubernetes API server|All
TCP|Inbound|2379-2380|etcd server client API|kube-apiserver, etcd
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|10251|kube-scheduler|Self
TCP|Inbound|10252|kube-controller-manager|Self

*Toda porta marcada por * é customizável, você precisa se certificar que a porta alterada também esteja aberta.*

**Workers**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|30000-32767|NodePort|Services All

## Conceitos-chave do k8s

É importante saber que o k8s gerencia contêineres de forma diferente de outros orquestradores, como Docker Swarm. Ele não trata contêineres diretamente, mas os organiza dentro de *pods*. Vamos aos principais conceitos:

- **Pod**: menor objeto do k8s. Agrupa um ou mais contêineres que compartilham recursos.
- **Deployment**: garante réplicas e gerencia o ciclo de vida das aplicações.
- **ReplicaSet**: mantém a quantidade desejada de pods em execução.
- **Services**: expõem aplicações via ClusterIP, NodePort ou LoadBalancer.
- **Volumes**: armazenamento compartilhado/persistente para pods.
- **Probes**: checagens de saúde (*liveness*, *readiness*, *startup*).
- **Ingress**: regras HTTP/HTTPS para expor serviços com host/path/TLS.
- **Secrets**: dados sensíveis (tokens, senhas, chaves) em base64.
- **ConfigMap**: configurações não sensíveis em chave/valor.

---

Nos próximos posts da série, vou detalhar **pods, deployments e services**, com exemplos práticos e manifests reais.
