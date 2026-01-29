---
title: "Série k8s: Fundamentos de Kubernetes"
date: 2026-01-29T09:00:00-03:00
description: "Início da série de Kubernetes: conceitos base, runtimes, componentes e visão geral da arquitetura."
tags: ["k8s", "kubernetes", "fundamentos", "containers", "cloud"]
categories: ["k8s", "cloud"]
draft: false
cover:
  image: "images/k8s-fundamentos.png"
  alt: "Ilustração sobre fundamentos de Kubernetes"
---

# Série k8s: Fundamentos de Kubernetes

Esta é a primeira parte de uma série de posts sobre Kubernetes. A ideia é construir uma base sólida, do básico ao avançado, passando por arquitetura, workloads, rede, storage, segurança, observabilidade e boas práticas de operação.

Se você está começando agora, este post é o ponto de partida. Se já usa Kubernetes no dia a dia, use como revisão rápida dos conceitos fundamentais e como referência quando surgir dúvida.

## Sobre esta série

Nesta série vamos abordar, de forma direta e prática:

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

Kubernetes é um orquestrador de contêineres: ele decide **onde** executar cada aplicação, **como** manter tudo saudável e **como** escalar quando necessário. Pense nele como o “sistema operacional” do seu cluster.

O projeto foi desenvolvido pela Google, em meados de 2014, como *open source*, com base no aprendizado do projeto Borg. Alguns outros produtos, como Apache Mesos e Cloud Foundry, também surgiram dessa mesma linha de pesquisa.

Como Kubernetes é uma palavra difícil de se pronunciar e escrever, a comunidade apelidou de **k8s**, seguindo o padrão i18n (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se “kates”.


### Alguns sites que devemos visitar

Abaixo estão os sites oficiais do projeto:

- https://kubernetes.io
- https://github.com/kubernetes/kubernetes/
- https://github.com/kubernetes/kubernetes/issues

E aqui estão as páginas oficiais das certificações (CKA, CKAD e CKS):

- https://www.cncf.io/certification/cka/
- https://www.cncf.io/certification/ckad/
- https://www.cncf.io/certification/cks/

## O container engine

Antes de aprofundar no Kubernetes, vale entender algumas peças do ecossistema. Uma delas é o **container engine**.

O *container engine* gerencia imagens, volumes e redes, garantindo o isolamento dos recursos que os containers usam (CPU, memória, storage, rede, etc.). É a camada que executa containers no dia a dia.

Hoje existem várias opções de *container engine*, que até pouco tempo atrás eram quase sinônimo de Docker.

Opções como Docker, CRI-O e Podman são bem conhecidas e preparadas para ambiente produtivo. O Docker é o mais popular e utiliza o containerd como runtime.

**Container runtime?** O que é isso?

Calma que vou explicar já já, mas antes temos que falar sobre a OCI. :)

### OCI — Open Container Initiative

A OCI é uma organização sem fins lucrativos que define padrões para containers, garantindo que funcionem em qualquer ambiente. Foi fundada em 2015 por Docker, CoreOS, Google, IBM, Microsoft, Red Hat e VMware, e hoje faz parte da Linux Foundation.

O principal projeto da OCI é o *runc*, um runtime de baixo nível usado por diferentes *container engines*, como o Docker.

O *runc* é open source, escrito em Go, e seu código está disponível no GitHub.

Agora sim podemos falar sobre o que é o container runtime.

### O container runtime

Para executar containers nos nós, é necessário ter um *container runtime* instalado em cada um deles. É ele quem “roda” o container de fato.

Quando você usa Docker ou Podman, está usando um *container runtime* por trás. Em outras palavras: o engine faz a gestão, e o runtime executa.

Existem quatro tipos comuns de *container runtime*:

- **Low-level**: são os runtimes “de base”, que conversam diretamente com o kernel e criam os processos dos containers. Exemplos: runc, crun e runsc.
- **High-level**: são camadas que gerenciam o ciclo de vida do container e usam um runtime low-level por baixo. Exemplos: containerd, CRI-O e Podman.
- **Sandbox**: adicionam uma camada extra de isolamento para segurança (útil em ambientes multi-tenant). O gVisor é um exemplo desse tipo.
- **Virtualized**: executam containers dentro de VMs leves para aumentar isolamento. A segurança é maior, mas há custo de performance. O Kata Containers é um exemplo.


## Arquitetura do k8s

Assim como outros orquestradores, o k8s segue o modelo **control plane + workers**. O control plane gerencia o cluster; os workers executam as aplicações. Em produção, recomenda-se ao menos três nós de control plane para alta disponibilidade.

É possível rodar um cluster em um único nó, mas apenas para estudos e labs.

Se você quiser usar Kubernetes localmente, existem várias opções que criam um cluster usando VMs ou Docker. Exemplos:

- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start): cluster Kubernetes com containers Docker. Útil para estudos, desenvolvimento e testes. **Não usar em produção**.
- [Minikube](https://github.com/kubernetes/minikube): cluster local com um nó. **Não usar em produção**.
- [MicroK8s](https://microk8s.io): pode ser usado em produção, especialmente para Edge e IoT.
- [k3s](https://k3s.io): distribuição leve, executa inclusive em Raspberry Pi.
- [k0s](https://k0sproject.io): distribuição em binário único, focada em simplicidade. **Pode ser usada em produção**.




### Componentes principais
Os componentes do Kubernetes se dividem em **control plane** e **workers**. O control plane toma decisões globais (como agendamento) e reage a eventos do cluster; os workers executam as cargas.

**Control plane**

- **kube-apiserver**: é o “front door” do cluster. Recebe todas as requisições do `kubectl` e dos demais componentes. Valida, autentica e autoriza as chamadas antes de gravar ou consultar estado no etcd.
- **etcd**: banco chave‑valor distribuído que guarda **todo o estado do cluster** (objetos, configurações e status). É crítico para recuperação de desastres; por isso backup é obrigatório em produção.
- **kube-scheduler**: escolhe em qual nó um Pod vai rodar. Ele avalia recursos disponíveis, afinidades, tolerations, políticas e outras restrições para decidir o melhor nó.
- **kube-controller-manager**: executa controladores que garantem o estado desejado do cluster. Ex.: se um Deployment quer 3 réplicas e só existem 2, o controller cria a terceira.
- **cloud-controller-manager**: integra o cluster ao provedor de nuvem. Ele cria e gerencia recursos externos (load balancers, rotas, volumes) usando a API do cloud provider.

**Workers**

- **kubelet**: agente em cada nó que “materializa” os Pods. Ele conversa com o runtime, inicia containers, monitora saúde e reporta status ao control plane.
- **kube-proxy**: implementa regras de rede para Services. Ele cria o encaminhamento (iptables/ipvs) que faz o tráfego chegar ao Pod correto.

![Arquitetura do Kubernetes]({{< relURL "images/k8s-arch.png" >}})

## Portas que devemos nos preocupar

**Control plane**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|6443*|Kubernetes API server|All
TCP|Inbound|2379-2380|etcd server client API|kube-apiserver, etcd
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|10251|kube-scheduler|Self
TCP|Inbound|10252|kube-controller-manager|Self

*Toda porta marcada por * é customizável. Se você alterar, garanta que a porta também esteja liberada no firewall.*

**Workers**

Protocol|Direction|Port Range|Purpose|Used By
--------|---------|----------|-------|-------
TCP|Inbound|10250|Kubelet API|Self, Control plane
TCP|Inbound|30000-32767|NodePort|Services All

## Conceitos-chave do k8s

O k8s não gerencia containers diretamente; ele organiza tudo dentro de **Pods**. Abaixo estão os conceitos mais usados no dia a dia, com uma explicação direta do “para quê” de cada um:

- **Pod**: menor unidade do k8s. Agrupa um ou mais containers que compartilham rede, volumes e ciclo de vida. Se o Pod morrer, ele é recriado.
- **Deployment**: forma padrão de rodar aplicações. Garante número de réplicas, faz rollouts/rollbacks e mantém a aplicação disponível durante atualizações.
- **ReplicaSet**: é o motor por trás do Deployment. Ele garante que a quantidade desejada de Pods esteja sempre rodando.
- **Service**: cria um endereço estável para acessar Pods. Pode expor internamente (ClusterIP) ou externamente (NodePort/LoadBalancer).
- **Volume**: define como dados são armazenados e compartilhados entre containers. Pode ser temporário (emptyDir) ou persistente (PV/PVC).
- **Probes**: checagens de saúde dos containers. *Liveness* reinicia, *readiness* controla se recebe tráfego e *startup* dá tempo extra para inicialização.
- **Ingress**: camada HTTP/HTTPS que define regras de entrada para múltiplos serviços, usando hosts, paths e TLS.
- **Secret**: guarda dados sensíveis (tokens, senhas, chaves). Evita hardcode em manifestos.
- **ConfigMap**: guarda configurações não sensíveis em chave/valor para aplicações.

---

Nos próximos posts da série, vou detalhar **pods, deployments e services**, com exemplos práticos e manifests reais.
