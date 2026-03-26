---
title: "O que realmente precisa existir em um módulo Terraform corporativo"
slug: "terraform-modulos-corporativos"
aliases:
  - "/posts/terraform-modules/"
date: 2026-03-12T10:30:00-03:00
description: "O que diferencia um modulo Terraform generico de um modulo corporativo e quais decisoes devem ficar encapsuladas para garantir padronizacao, seguranca e governanca."
summary: "O que diferencia um modulo Terraform generico de um modulo corporativo e quais decisoes devem ficar encapsuladas para garantir padronizacao, seguranca e governanca."
tags: ["terraform", "iac", "aws", "modulos", "governanca", "seguranca", "engenharia-de-plataforma", "vpc", "rds", "ec2", "opa"]
categories: ["terraform", "governanca", "cloud"]
draft: false
cover:
  image: "images/terraform-module.png"
  alt: "Ilustracao abstrata sobre modulos Terraform corporativos"
---

> Um módulo genérico cria infraestrutura. Um módulo corporativo entrega infraestrutura que funciona dentro da sua organização.

Qualquer pessoa cria uma VPC na AWS em 10 minutos. O Terraform Registry tem módulos prontos, bem documentados, amplamente usados.

O problema não é criar o recurso. É garantir que ele funcione dentro da organização — com a nomenclatura correta, integrado à rede certa, com segurança aplicada, padrões respeitados e rastreabilidade desde o primeiro apply.

Na minha experiência, a diferença entre um módulo genérico e um módulo corporativo é exatamente isso: o módulo corporativo carrega as decisões da organização. Quem o consome não precisa conhecê-las — só precisa passar as variáveis essenciais.

---

## TL;DR

Um módulo corporativo não é apenas código reutilizável.
Ele encapsula as decisões arquiteturais da organização e garante que qualquer recurso criado já nasça dentro dos padrões operacionais, de segurança e governança. É onde a arquitetura da organização se materializa em código.

Quando bem construído, ele garante que qualquer recurso criado na organização nasce padronizado, integrado e seguro — independente de quem o criou ou quando.

---

## O problema com módulos genéricos

Um módulo genérico resolve o problema técnico. Cria o recurso, configura o básico, faz o que promete.

O que ele não sabe é como a sua organização funciona:

- Qual o padrão de nomenclatura dos recursos
- Quais configurações de segurança são obrigatórias
- Como o recurso se integra com o restante da organização
- Quais tags são necessárias para rastreabilidade e FinOps
- Quais decisões de arquitetura já foram tomadas e não devem variar

Quando esse conhecimento não está no módulo, ele fica na cabeça das pessoas. E conhecimento na cabeça das pessoas não escala, não é auditável e não é consistente.

---

## O que um módulo corporativo precisa ter

Os exemplos abaixo usam VPC, RDS e EC2 para ilustrar o conceito. A lógica se aplica a qualquer módulo da organização.

### VPC — exemplo central

Uma VPC corporativa não entrega só uma rede. Ela entrega uma rede já integrada na organização:

- **Nomenclatura padronizada** — subnets, route tables e security groups nascem com o padrão de naming da empresa, sem variação entre times ou datas
- **Estrutura de subnets** — quantas AZs, como são divididas entre pública, privada e isolada, padrão de CIDR por camada
- **Roteamento definido** — saída de internet via NAT Gateway ou Internet Gateway dependendo do ambiente, já configurada no módulo
- **Transit Gateway** — attachment criado automaticamente, com as rotas para comunicação com outras VPCs da organização
- **DNS e DHCP** — DHCP Options Set com domínio interno e resolvers corretos, Route 53 Resolver com regras de forward para o DNS centralizado

```hcl
module "vpc" {
  source = "git::https://github.com/org/terraform-module-vpc.git?ref=v2.3.0"

  name        = "payments"
  environment = var.environment
  cidr        = "10.20.0.0/16"

  # o restante vem do módulo:
  # - subnets com nomenclatura padrão
  # - attachment no Transit Gateway
  # - DHCP Options com domínio interno
  # - Route 53 Resolver rules
  # - roteamento para a organização
}
```

### RDS — segurança e resiliência como padrão

Um módulo corporativo de RDS não deixa decisões críticas para quem consome. Elas já vêm definidas:

- **Criptografia** habilitada por padrão — em repouso e em trânsito
- **Backup automático** com retenção mínima definida pela organização
- **Multi-AZ** habilitado em produção automaticamente, baseado no ambiente
- **Parâmetros de segurança** — versão mínima de TLS, acesso restrito por security group padrão
- **Snapshot final obrigatório** em deleção, protegido contra remoção acidental

Quem cria um banco de dados passa o engine, o tamanho e o ambiente. O módulo garante que as decisões de resiliência e segurança já estão aplicadas.

### EC2 — consistência operacional

Um módulo corporativo de EC2 vai além do tipo de instância e da AMI. Ele garante que toda máquina criada na organização nasce operacionalmente pronta:

- **AMI corporativa** — imagem base aprovada, já com hardening aplicado
- **User data padronizado** — instalação de agentes de monitoramento, ferramentas de gerenciamento e configurações de segurança executadas automaticamente na inicialização
- **Integração com SSM Patch Manager**
- **Agentes de segurança e observabilidade** instalados na inicialização
- **Tags obrigatórias** — para rastreabilidade, FinOps e inventário
- **IAM Instance Profile padrão** — permissões mínimas necessárias para operação

Uma EC2 criada com o módulo já nasce com tudo que a organização precisa para operar aquela máquina — sem depender de configuração manual pós-provisionamento.

---

## O que pertence ao módulo e o que pertence às variáveis

O critério é simples: decisões que garantem que o recurso funciona dentro da organização e segue os padrões da empresa ficam no módulo. Decisões que variam por aplicação ou contexto vão nas variáveis.

**Pertence ao módulo — não é negociável:**
- Padrão de nomenclatura
- Integrações organizacionais (Transit Gateway, DNS, SSM)
- Configurações mínimas de segurança
- Tags obrigatórias
- Decisões de resiliência padrão

**Pertence às variáveis — decisão de quem consome:**
- Nome e ambiente
- Tamanho do recurso
- CIDRs e configurações específicas da aplicação
- Tags adicionais
- Overrides explicitamente permitidos pelo módulo

### Mesmo o configurável pode ter limites

Expor uma variável não significa abrir mão de governança. Mesmo o que é configurável pode ser controlado por política.

OPA + Conftest permite escrever regras que validam o plan do Terraform antes do apply — garantindo que, mesmo quando há liberdade de configuração, certos limites não sejam ultrapassados:

```rego
# não permitir instâncias RDS sem Multi-AZ em produção
deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_db_instance"
  resource.change.after.tags.environment == "prod"
  not resource.change.after.multi_az
  msg := sprintf("RDS sem Multi-AZ em produção: %v", [resource.address])
}

# não permitir EC2 sem SSM habilitado
deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_instance"
  not resource.change.after.iam_instance_profile
  msg := sprintf("EC2 sem Instance Profile para SSM: %v", [resource.address])
}
```

Isso cria uma camada de governança que complementa o módulo: o módulo define os padrões, o OPA garante que eles não foram contornados — seja por configuração manual, seja por variável fora do esperado.

---

## Versionamento: o módulo precisa evoluir de forma controlada

Módulo sem versionamento é módulo sem governança.

Cada módulo corporativo deve ter seu próprio repositório — separado do repositório de infraestrutura que o consome. Essa separação garante que cada módulo tenha seu próprio ciclo de vida, seu próprio processo de revisão e seu próprio histórico de mudanças:

```text
github.com/org/terraform-module-vpc
github.com/org/terraform-module-rds
github.com/org/terraform-module-ec2
```

Quando o source aponta para uma branch sem versão fixada, qualquer atualização afeta todos os consumidores imediatamente — sem aviso, sem revisão, sem controle.

Com versionamento explícito por tag, cada repositório de infra declara exatamente qual versão do módulo está usando:

```hcl
# dev validando nova versão
module "vpc" {
  source = "git::https://github.com/org/terraform-module-vpc.git?ref=v2.4.0"
  ...
}

# prod ainda na versão estável
module "vpc" {
  source = "git::https://github.com/org/terraform-module-vpc.git?ref=v2.3.0"
  ...
}
```

Dev pode validar `v2.4.0` enquanto prod continua em `v2.3.0`. A adoção de qualquer mudança é uma decisão explícita — visível, rastreável e reversível. Times diferentes podem evoluir no próprio ritmo, sem forçar atualizações em toda a organização ao mesmo tempo.

---

## Segurança como consequência

Quando toda a infraestrutura nasce a partir de módulos versionados, a correção de vulnerabilidades segue um caminho claro:

1. Vulnerabilidade identificada — por exemplo, EC2s sendo criadas sem o agente de segurança instalado
2. Correção feita no módulo, nova versão publicada
3. Novos recursos já nascem corrigidos — o problema é estancado na origem
4. Para o legado: **quem está abaixo da versão corrigida?** Você tem a lista

Sem módulos versionados, essa mesma pergunta vira uma investigação manual do ambiente inteiro.

---

## O módulo como contrato entre times

Em organizações com múltiplos times, o módulo corporativo resolve um problema que vai além da tecnologia.

Sem módulos bem definidos, o time de plataforma vira gargalo. Toda nova conta, todo novo recurso, todo novo ambiente precisa passar por quem conhece as decisões arquiteturais da organização.

Com módulos corporativos, o time de plataforma define o padrão uma vez. Os times de produto consomem com autonomia — sem precisar entender como o Transit Gateway funciona, sem precisar configurar DNS, sem precisar saber quais agentes instalar em cada EC2.

O resultado é **autonomia com guardrails**: times se movem rápido dentro de um espaço bem definido, e a governança é consequência natural do processo.

---

## Conclusão

A diferença entre um módulo genérico e um módulo corporativo não está no código em si. Está no que ele carrega.

Um módulo corporativo bem construído entrega mais do que infraestrutura — entrega infraestrutura que já nasce integrada, padronizada e segura dentro da organização. Quem o consome não precisa tomar as mesmas decisões repetidamente. Essas decisões já foram tomadas, estão versionadas e evoluem de forma controlada.

Quanto mais a organização cresce, mais esse investimento se paga.

---
