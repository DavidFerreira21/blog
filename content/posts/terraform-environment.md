---
title: "Terraform multi-environment: decisões que evitam que sua infraestrutura vire caos"
date: 2026-03-11T10:30:00-03:00
description: "As decisões essenciais para estruturar ambientes Terraform com consistência, governança, rastreabilidade e controle de drift."
tags: ["terraform", "iac", "devops", "aws", "governanca", "modulos", "pipeline", "multiambiente", "engenharia-de-plataforma"]
categories: ["terraform", "devops", "nuvem", "governanca"]
draft: false
cover:
  image: "images/terraform-mult.png"
  alt: "Ilustração abstrata sobre arquitetura e ambientes Terraform"
---

> Não existe uma única forma certa de estruturar ambientes Terraform. Mas existem decisões que, se ignoradas, cobram o preço mais tarde.

Quando um time começa a trabalhar com Terraform, o foco é entregar. Um ambiente, um repositório, tudo funcionando. Simples.

O problema aparece quando o segundo ambiente chega. E o terceiro. E quando times diferentes começam a tocar na mesma infraestrutura.

Na minha experiência, alguns padrões se repetem — e o custo de não endereçá-los cedo também. Este post reúne as decisões que, na prática, mais impactaram a forma como ambientes Terraform evoluem. Não existe uma resposta única para cada uma delas. O contexto, a maturidade do time e as ferramentas disponíveis influenciam o caminho. Mas ignorá-las costuma cobrar o preço mais tarde.

Em organizações maiores, essas decisões deixam de ser apenas uma questão de organização de código. Elas passam a fazer parte da engenharia de plataforma: a forma como ambientes são estruturados define como equipes entregam infraestrutura, como padrões são aplicados e como riscos operacionais são controlados.

---

## TL;DR

Independente de como você implementa, toda estrutura de ambientes Terraform precisa responder bem a quatro pilares:

- **Design** — o código é consistente entre ambientes?
- **Governança** — quem pode fazer o quê, e como os padrões são aplicados?
- **Rastreabilidade** — o que foi aplicado, quando, por quem e com qual código?
- **Drift** — quando o ambiente diverge do código, você sabe?

Cada decisão que você toma ao estruturar seus ambientes vai fortalecer ou enfraquecer esses pilares.

---

## O problema que aparece com o tempo

A solução mais comum quando o segundo ambiente aparece é simples: copiar a pasta existente e ajustar algumas variáveis.

```text
infra-dev/
infra-prod/
```

Isso funciona… até não funcionar mais.

- Correções feitas em prod nunca chegam em dev
- Ambientes começam a divergir silenciosamente
- Ninguém tem certeza de qual versão da infraestrutura está rodando

Infraestrutura vira um conjunto de variações não intencionais. Variação não controlada em infraestrutura vira complexidade acumulada — e complexidade acumulada vira risco operacional.

---

## Decisão 1: como separar os ambientes?

Essa é a primeira decisão estrutural. E ela define boa parte do que vem depois.

Existem abordagens diferentes — pastas separadas por ambiente, workspaces do Terraform, repositórios distintos. Cada uma tem trade-offs.

### O que considerar

A separação precisa garantir que dev e prod não interferem um no outro — nem no código, nem no state, nem nas credenciais. Ao mesmo tempo, precisa garantir que os dois rodam o mesmo código. Se a separação exige manter código duplicado, ela vai gerar divergência.

Uma abordagem que equilibra bem esses dois pontos é um único repositório com código compartilhado e variáveis separadas por ambiente, com a identificação do ambiente feita pela branch da pipeline.

```text
repo-infra/
├── main.tf
├── variables.tf
├── versions.tf
└── envs/
    ├── dev.tfvars
    ├── hml.tfvars
    └── prod.tfvars
```

Todos os ambientes executam exatamente o mesmo código. O que muda são os valores — tamanho de instâncias, número de réplicas, CIDRs, retenção de logs.

**Exceções existem.** Alguns recursos precisam de tratamento específico por ambiente. Isso pode ser resolvido com variáveis condicionais ou `data sources`. O importante é que exceções sejam realmente exceções — quanto mais genérico for o código, maior a consistência e a governança.

---

## Decisão 2: onde ficam os módulos?

Módulos são onde as decisões arquiteturais da organização vivem. Essa decisão define como elas vão evoluir.

### O que considerar

Módulos dentro do repositório de infra são convenientes, mas criam acoplamento. Uma mudança no módulo afeta imediatamente todos os ambientes — sem revisão, sem controle.

Módulos em repositórios próprios, com versionamento explícito, mudam esse cenário:

```hcl
module "vpc" {
  source = "git::https://github.com/org/terraform-module-vpc.git?ref=v2.3.0"
  ...
}
```

Dev pode testar `v2.4.0` enquanto prod ainda roda `v2.3.0`. A adoção de qualquer mudança é uma decisão explícita — visível, rastreável e reversível.

Vale destacar um ponto importante: módulos corporativos não são apenas código reutilizável. Eles são o lugar onde a arquitetura da organização vive — padronização de nomenclatura, tags obrigatórias, configurações de segurança, decisões de rede. A forma como uma VPC é criada, como ela se conecta à organização, quais rotas existem por padrão — tudo isso pode e deve estar no módulo, não espalhado por cada repositório de infra. Quem consome o módulo herda essas decisões automaticamente, sem precisar conhecê-las em detalhe.

**O impacto vai além da organização.** Quando uma vulnerabilidade de segurança é corrigida no módulo, todos que adotarem a nova versão já nascem corrigidos. Para o legado, a pergunta é objetiva: quem ainda está na versão antiga? Você tem a lista — sem precisar investigar o ambiente inteiro.

Sem versionamento, essa mesma pergunta vira uma investigação.

---

## Decisão 3: onde e como o state é gerenciado?

O state é onde o Terraform mantém o mapa da infraestrutura real. Uma decisão errada aqui pode causar desde inconsistências até destruição acidental de recursos.

### O que considerar

Cada ambiente precisa do seu próprio state. Misturar ambientes no mesmo state aumenta o risco de impacto acidental — um apply em dev pode afetar prod.

Uma estrutura que funciona bem é manter o state centralizado em uma conta dedicada — shared services — separado por prefixo:

```text
tfstate-org/
├── dev/app01/terraform.tfstate
├── hml/app01/terraform.tfstate
├── preprod/app01/terraform.tfstate
└── prod/app01/terraform.tfstate
```

O backend não deve ficar hardcoded no código. Com partial configuration, os valores são passados pela pipeline em tempo de execução — o código não sabe qual ambiente está rodando. Quem sabe é a pipeline.

O lock — para evitar applies concorrentes no mesmo ambiente — pode ser feito via DynamoDB ou pelo recurso nativo de locking do S3, disponível nas versões mais recentes do Terraform.

---

## Decisão 4: quem controla quando e como o apply acontece?

Essa decisão define a rastreabilidade da sua infraestrutura. Apply manual, direto da máquina do engenheiro, remove qualquer possibilidade de auditoria.

### O que considerar

Infraestrutura deve ser aplicada via pipeline. Não por rigidez, mas porque pipeline é o único mecanismo que garante que toda mudança passou por validação, foi revisada e foi registrada.

Um fluxo de promoção entre ambientes torna isso explícito:

```text
feature → dev → hml → preprod → prod
```

Cada merge dispara a pipeline no ambiente correspondente. Prod exige aprovação manual antes do apply.

Em ambientes mais maduros, o plan gerado na pipeline também pode ser tratado como um artefato imutável — o mesmo plan aprovado é o que será aplicado em produção.

O fluxo é o padrão, mas pode ser adaptado. Se uma aplicação só tem dois ambientes, as etapas do meio não existem. O que não se pode abrir mão é o princípio: **mudanças passam pela pipeline**.

A pipeline também é onde a segurança é verificada — ferramentas como Checkov cobrem boas práticas e benchmarks conhecidos. Para políticas específicas da organização, OPA + Conftest permite escrever regras customizadas em cima do plan do Terraform.

---

## Decisão 5: o que você vai fazer quando o ambiente divergir?

Drift é inevitável em qualquer ambiente real. Troubleshooting de madrugada, intervenção emergencial, mudança manual que "era só temporária". Acontece.

### O que considerar

A questão não é se o drift vai acontecer — é se você vai saber quando ele aconteceu.

Sem um processo para detectar drift, o ambiente começa a divergir silenciosamente do código. Com o tempo, ninguém sabe mais qual é a fonte da verdade.

Uma prática simples é executar `terraform plan` periodicamente — não para aplicar, só para detectar divergências entre o state e o ambiente real. Algumas organizações automatizam isso com pipelines programadas que alertam quando o plan retorna diferenças.

Detectar drift cedo é muito mais barato do que descobrir em produção.

---

## Anti-patterns que criam problemas em escala

Alguns padrões parecem funcionar no início, mas cobram o preço mais tarde:

- **Copiar infraestrutura por ambiente** — gera divergência inevitável. O código de dev e prod vão divergir, e ninguém vai saber exatamente quando ou como.
- **State compartilhado entre ambientes** — aumenta o risco de impacto acidental. Um erro de configuração pode afetar o ambiente errado.
- **Apply manual** — executado diretamente da máquina do engenheiro, remove rastreabilidade e dificulta auditoria. Sem pipeline, não há como saber o que foi aplicado, quando e por quem.
- **Módulos sem versionamento** — qualquer atualização no módulo afeta todos os consumidores imediatamente. Mudanças chegam sem aviso e sem controle.
- **Segredos em tfvars** — variáveis de ambiente não são o lugar para segredos. Use AWS Secrets Manager, Parameter Store ou Vault.

---

## Conclusão

Não existe uma estrutura de ambientes Terraform que funcione para todo mundo. O contexto importa — o tamanho do time, a maturidade da organização, as ferramentas disponíveis.

Se as decisões discutidas aqui forem tomadas de forma consistente, os quatro pilares mencionados no início — design, governança, rastreabilidade e visibilidade de drift — passam a se reforçar mutuamente.

A infraestrutura deixa de depender de conhecimento implícito e passa a ser operada de forma previsível.
