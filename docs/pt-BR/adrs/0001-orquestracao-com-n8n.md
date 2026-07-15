# ADR-0001: Usar n8n para Orquestração de Workflows

> 🇺🇸 [Read in English](../../en/adrs/0001-n8n-orchestration.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

O sistema precisa: receber webhooks do WhatsApp, enfileirar e processar mensagens, chamar um LLM com suporte a ferramentas, integrar com Google Calendar, publicar no Chatwoot, rodar jobs agendados (lembretes, health checks, retries) e alertar sobre falhas — tudo mantido por uma única pessoa, sem equipe de operações dedicada. A escolha era entre escrever isso à mão como serviços de backend (Node/Python) ou construir sobre uma plataforma visual de orquestração de workflows.

## Decisão

Usar n8n, self-hosted via Docker, como camada de orquestração para tudo acima.

## Alternativas consideradas

- **Serviços de backend customizados (Node.js/Python) com uma fila de jobs (ex: BullMQ)**: controle total e melhor testabilidade, mas cada integração (provedor de WhatsApp, Calendar, Chatwoot, chamada de ferramentas do LLM, agendamento, lógica de retry) precisaria ser construída e mantida do zero, e cada mudança exigiria um deploy.
- **Outra plataforma de automação (ex: Zapier, Make)**: mais rápida para começar, mas o preço de SaaS escala mal com volume de mensagens, e nenhuma das duas oferece o mesmo nível de controle sobre credenciais self-hosted, workflows com error trigger ou acesso em nível de banco de dados que este sistema precisa.
- **n8n (escolhido)**: workflows visuais, agendamento e error triggers nativos, uma grande biblioteca de nós de integração prontos, e possibilidade de self-host — evitando tanto o custo de "construir tudo do zero" quanto o custo de "pagar por execução em escala" das ferramentas de automação SaaS.

## Consequências

**Positivas**:
- Muito mais rápido para construir e iterar sobre lógica cheia de integrações do que código de backend escrito à mão, o que importou para uma construção solo sob pressão de tempo.
- O fluxo de controle do sistema é visualmente auditável — útil tanto para a própria depuração do mantenedor quanto para explicar o sistema a um stakeholder não-técnico.
- Armazenamento de credenciais, agendamento e tratamento de erro via Error Trigger nativos removeram infraestrutura que, de outra forma, precisaria ser construída.

**Negativas**:
- Lógica de workflow é mais difícil de testar unitariamente do que código puro — a corretude depende mais de testes ponta a ponta (staging + allowlist) do que de cobertura de testes automatizada.
- JSON de workflow exportado produz diffs ruidosos, tornando revisão de código e histórico de mudanças menos legíveis do que um pull request típico.
- Este deploy Docker específico não suporta variáveis `$env.*` dentro dos workflows, forçando todos os segredos para dentro do armazenamento de credenciais do próprio n8n em vez de configuração via variável de ambiente padrão.
- A duplicação staging/produção dos workflows de maior risco (em vez de um único workflow parametrizado) é um custo de manutenção que cresce com o número de workflows — rastreado como débito arquitetural (ver [Arquitetura § 9](../arquitetura.md#9-débito-arquitetural-conhecido)).

## Relacionados

[Arquitetura](../arquitetura.md), [Integrações § 3](../integracoes.md#3-n8n-orquestração)
