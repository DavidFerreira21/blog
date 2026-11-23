---
title: "Kubernetes Visual Guide: aprendendo Kubernetes com diagramas, desafios e laboratórios práticos"
slug: "kubernetes-visual-guide"
date: 2026-11-23T10:00:00-03:00
description: "Como minhas anotações de Kubernetes evoluíram para um guia open source com 49 aulas, infográficos e desafios práticos com Kind."
summary: "De anotações espalhadas a uma trilha visual e prática de Kubernetes: conheça a proposta do Kubernetes Visual Guide e como começar."
tags: ["kubernetes", "kind", "gitops", "cloud-native", "open-source", "aprendizado"]
categories: ["kubernetes", "projetos"]
draft: true
cover:
  image: "images/k8s-arch.png"
  alt: "Diagrama de arquitetura do Kubernetes com componentes do cluster"
---

Há um tempo, eu mantinha um repositório com anotações sobre Kubernetes. Era o tipo de material que a gente vai acumulando enquanto estuda ou resolve problemas: um YAML aqui, um comando útil ali, explicações de conceitos, links para documentação e alguns diagramas.

O conteúdo era útil para mim, mas tinha um problema: **estava desorganizado**. Quando precisava revisar um assunto, gastava tempo tentando encontrar o que tinha escrito. E, para alguém de fora, seguir aquelas anotações como uma sequência de aprendizado seria ainda mais difícil.

Comecei a revisar e organizar esse material pensando em facilitar minhas próprias consultas. No processo, percebi uma oportunidade: **por que não transformar essas anotações em uma trilha que também pudesse ajudar outras pessoas a aprender Kubernetes?**

Foi assim que nasceu o [Kubernetes Visual Guide](https://github.com/DavidFerreira21/kubernetes-visual-guide), um projeto open source que reúne conteúdo visual, explicações técnicas e exercícios práticos.

## O que eu queria resolver

A documentação oficial do Kubernetes é excelente e continua sendo uma referência essencial. Mas, quando estamos aprendendo, nem sempre basta entender um objeto isoladamente.

Saber o que é um Pod não significa necessariamente compreender o que acontece quando ele falha. Conseguir escrever um Deployment não garante que ficou claro como ele se relaciona com ReplicaSets. E aplicar um manifesto com sucesso não significa que a aplicação está funcionando.

Eu queria uma forma de estudar que respondesse a três perguntas:

- **O que é isso e qual problema resolve?**
- **O que acontece de verdade dentro do Kubernetes?**
- **Como posso observar esse comportamento em um cluster?**

A partir dessas perguntas, comecei a reorganizar o conteúdo em aulas, infográficos e laboratórios.

## Como o guia está organizado

O projeto tem atualmente **49 aulas, 49 infográficos e 49 roteiros de laboratório**, distribuídos em quatro camadas.

### 1. Fundamentos e recursos essenciais

A primeira camada começa pela arquitetura básica e avança por Pods, Deployments, Services, Namespaces, ConfigMaps, Secrets, Probes, Volumes, Ingress e HPA.

É a base para entender como declarar, executar, expor e acompanhar workloads.

### 2. Operação, segurança e governança

A segunda camada entra em RBAC, ServiceAccounts, políticas de segurança, quotas, scheduling, disponibilidade, NetworkPolicy e troubleshooting.

Aqui a preocupação deixa de ser apenas criar recursos: precisamos entender **por que uma operação é permitida, negada, agendada ou bloqueada**.

### 3. Arquitetura e infraestrutura do Kubernetes

Na terceira camada, o foco é entender o funcionamento interno do cluster: API Server, etcd, scheduler, kubelet, controllers, CRDs, rede, storage, Gateway API, Helm e Kustomize.

É a parte que ajuda a conectar os comportamentos observados às responsabilidades dos componentes.

### 4. Ferramentas do ecossistema

A quarta camada apresenta Argo CD, Kyverno, Cilium e Prometheus com Grafana, explorando GitOps, políticas, rede e observabilidade.

Esses exercícios exigem cuidados adicionais, porque instalar um CRD ou aplicar um YAML **não prova que um controller está instalado e reconciliando o recurso**.

## A principal diferença: aprender observando

Um dos pontos que mais quis preservar no projeto foi o aprendizado ativo.

Em vez de começar com a solução, cada laboratório apresenta um desafio. O aluno prepara o ambiente, completa o próprio manifesto YAML, aplica o recurso, observa o resultado e investiga o comportamento.

Um exemplo simples aparece logo no início.

Primeiro, o aluno cria um **Pod isolado**. Depois, exclui esse Pod e observa que ele não é recriado automaticamente.

Na aula seguinte, cria um **Deployment** com três réplicas. Ao excluir um dos Pods, observa que um novo aparece para manter a quantidade desejada.

O contraste é importante: **não é o arquivo YAML que recria o Pod**. O comportamento depende dos controladores responsáveis por reconciliar os recursos.

Esse tipo de experiência ajuda a construir um modelo mental do Kubernetes, não apenas a memorizar comandos.

## Laboratórios com Kind e um caminho para começar

Para permitir que as pessoas experimentem sem depender de uma conta cloud, os laboratórios usam principalmente o [Kind](https://kind.sigs.k8s.io/), que executa um cluster Kubernetes local em containers.

O repositório oferece um caminho de início, com instruções de ambiente e comandos padronizados. Por exemplo, depois de criar o cluster e ler a primeira aula:

```bash
make lab-prepare LAB=01.01

# Complete o YAML gerado em .labs/01.01/ seguindo o desafio.
kubectl apply -f .labs/01.01/

make lab-test LAB=01.01
make lab-observe LAB=01.01

# Execute o experimento e, ao terminar:
make lab-down LAB=01.01
```

O comando `lab-prepare` prepara o exercício sem aplicar a solução. A ideia é que a pessoa escreva o YAML, compare estado desejado e observado e consiga explicar o resultado.

Os laboratórios também incluem orientações de diagnóstico, critérios de conclusão e links para documentação oficial.

**Importante:** alguns cenários exigem addons, como Ingress Controller, Metrics Server, Cilium ou Argo CD. O projeto diferencia validação de manifesto e teste funcional, e os roteiros continuam em evolução. Não considero todos os 49 laboratórios funcionalmente homologados apenas porque seus arquivos e verificações estão disponíveis.

## Para quem esse material pode ser útil?

Pensei no Kubernetes Visual Guide principalmente para quem está começando a estudar Kubernetes, para quem já trabalha com clusters mas quer revisar conceitos específicos e para quem prefere entender mecanismos por meio de experimentos.

Para mim, ele também resolve a necessidade que deu origem a tudo: ter uma **base de consulta organizada e fácil de revisitar**.

Outro benefício de organizar esse material em um repositório público é poder melhorar o conteúdo ao longo do tempo, incluindo correções, exemplos e contribuições da comunidade.

## O que vem pela frente

O guia não é um material fechado. Quero continuar refinando a clareza das aulas, revisando os infográficos e ampliando as evidências dos laboratórios que dependem de componentes adicionais.

Minha intenção é manter o projeto útil tanto para uma primeira jornada de estudo quanto para consultas pontuais no dia a dia.

Se você está estudando Kubernetes, ou já usa a tecnologia e quer revisar algum conceito, fica o convite para conhecer:

**[Acessar o Kubernetes Visual Guide no GitHub](https://github.com/DavidFerreira21/kubernetes-visual-guide)**

No README você encontra o roteiro de primeiros passos e pode explorar as quatro camadas. Se encontrar algo que possa ser melhorado, sugestões, issues e contribuições são bem-vindas.

No fim, o projeto começou como uma forma de organizar minhas próprias anotações. Agora, a ideia é que ele também seja útil para quem está construindo seu caminho com Kubernetes.
