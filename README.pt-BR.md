# ZapCare — Estudo de Caso: Automação com IA para Clínicas

> 🇺🇸 Read in English: [README.md](README.md)

O **ZapCare** é um produto real, em operação, que automatiza o atendimento de pacientes de clínicas de pequeno e médio porte no Brasil via WhatsApp, usando uma assistente de IA conversacional ("Sofia"), uma camada de orquestração de workflows e um painel administrativo próprio.

Este repositório é um **estudo de caso**, não o código-fonte do produto. Ele documenta os requisitos, regras de negócio, arquitetura, decisões técnicas e trade-offs por trás de um sistema desenhado, construído e operado de ponta a ponta por uma única pessoa — cobrindo pensamento de produto, análise de requisitos, arquitetura de sistemas e desenvolvimento híbrido low-code/full-stack.

> Detalhes sensíveis (nomes reais de clínicas, números de telefone, IPs de infraestrutura, identificadores internos de banco de dados) foram anonimizados ou substituídos por placeholders em toda a documentação. Arquitetura, regras de negócio e decisões técnicas são reais e não foram alteradas. Tudo que ainda não foi confirmado contra o sistema em produção está marcado explicitamente como **`[a validar]`**.

---

## Sumário

- [Visão geral](#visão-geral)
- [O problema](#o-problema)
- [Público-alvo](#público-alvo)
- [A solução](#a-solução)
- [Principais funcionalidades](#principais-funcionalidades)
- [Stack tecnológica](#stack-tecnológica)
- [Status do projeto](#status-do-projeto)
- [Mapa da documentação](#mapa-da-documentação)
- [O que este projeto demonstra](#o-que-este-projeto-demonstra)
- [Contato](#contato)

---

## Visão geral

Clínicas no Brasil, em grande maioria, concentram o atendimento ao paciente em um único número de WhatsApp, operado manualmente por uma recepcionista. Isso funciona até o volume crescer — aí o tempo de resposta desaba, agendamentos se perdem no histórico da conversa, e ninguém tem visibilidade real do que está acontecendo em cada atendimento.

O ZapCare substitui esse gargalo manual por uma assistente de IA que conversa com pacientes em linguagem natural no mesmo número de WhatsApp da clínica, qualifica o lead, responde dúvidas frequentes, verifica disponibilidade na agenda, agenda consultas e transfere para um humano no exato momento em que a conversa precisa de um — enquanto um painel administrativo leve dá ao dono da clínica visibilidade e controle sem precisar tocar em terminal ou editor de workflow.

## O problema

- Clínicas perdem leads e pacientes por causa da demora na resposta via WhatsApp, especialmente fora do horário comercial.
- Uma recepcionista sozinha, dividindo atenção entre WhatsApp, agenda (papel ou Google Agenda) e atendimento presencial, não consegue escalar o volume de conversas sem contratar mais gente.
- Ferramentas prontas de chatbot ou são rígidas demais (bots de menu que frustram o paciente) ou genéricas demais (sem conhecimento da agenda, dos serviços ou das regras de agendamento da clínica).
- O dono da clínica não tem visibilidade operacional: sem fila de conversas pendentes, sem registro do que o bot respondeu, sem alerta quando algo quebra silenciosamente.

## Público-alvo

- **Primário**: clínicas independentes de estética, odontologia e especialidades no Brasil, com 1 a 5 profissionais, que já recebem agendamentos via WhatsApp e usam Google Agenda/Calendar.
- **Secundário (dentro do produto)**: a recepção da clínica (que assume as conversas que a Sofia escala) e o dono da clínica (que acompanha cobrança, conversas e agendamentos pelo painel).
- **Interno**: o próprio operador do ZapCare (dono de produto / plantão), que monitora saúde do sistema, status de cobrança e pipeline comercial.

## A solução

O ZapCare é um sistema em camadas, não um chatbot único:

1. **Sofia (a assistente de IA)** — um agente conversacional baseado em LLM (com roteamento entre um modelo rápido/barato e um modelo mais robusto conforme a complexidade da pergunta) que mantém contexto por conversa, responde dúvidas e chama ferramentas para checar disponibilidade e agendar consultas.
2. **Uma camada de orquestração** construída em n8n, responsável por receber os webhooks do WhatsApp, deduplicar e enfileirar mensagens, garantir o consentimento LGPD antes de qualquer resposta da IA, rotear para a Sofia ou para um humano, enviar lembretes e se automonitorar (health checks, alertas de SLA, retry e fila de falhas permanentes).
3. **Um backend relacional** (Supabase/PostgreSQL) com clínicas, conversas, fila de mensagens, agendamentos, registros de consentimento e logs estruturados — com Row-Level Security e RPCs `SECURITY DEFINER` usadas deliberadamente para manter os caminhos de escrita estreitos.
4. **Um caminho de transferência humana** para o Chatwoot, de forma que quando uma conversa precisa de uma pessoa, ela vira um ticket de suporte normal com contexto completo (um resumo gerado por IA), não uma transferência a frio.
5. **Um painel administrativo** (React/Vite/Supabase) para o operador do ZapCare gerenciar clínicas, status de cobrança, conversas, fila de processamento e um CRM comercial — sem precisar acessar o n8n ou o banco de dados diretamente.
6. **Uma landing page pública** com um fluxo interativo de qualificação de leads ("diagnóstico") que alimenta o CRM comercial.

Veja [`docs/pt-BR/arquitetura.md`](docs/pt-BR/arquitetura.md) para o diagrama completo do sistema e o detalhamento de cada componente.

## Principais funcionalidades

- Conversas em linguagem natural via WhatsApp, incluindo **transcrição de mensagens de áudio** (áudio → texto → resposta da IA).
- **Gate de consentimento LGPD**: nenhuma resposta da IA é gerada para um paciente novo até o consentimento explícito ser registrado.
- **Roteamento de IA em camadas**: perguntas simples vão para um modelo mais rápido/barato, perguntas complexas escalam para um modelo mais robusto — um trade-off deliberado de custo versus qualidade.
- **Agente que usa ferramentas**: a Sofia chama sub-workflows dedicados para checar disponibilidade e agendar consultas, em vez de "alucinar" lógica de agenda dentro do prompt.
- **Transferência humana com briefing**: quando a Sofia não pode ou não deve continuar (pedido explícito, caso sensível, pergunta fora de escopo, erro de agendamento), a conversa vai para o Chatwoot com um resumo escrito por IA, e a IA fica pausada naquela conversa até um humano resolver.
- **Lembretes automáticos** de consultas agendadas, com rastreamento de confirmação.
- **Fila de mensagens autorregenerativa**: retry com backoff exponencial, um caminho de dead-letter após falhas repetidas, e alerta ao operador.
- **Automonitoramento operacional**: health checks agendados (integrações, fila acumulada, volume de envio) e detecção de estouro de SLA em transferências humanas paradas, ambos alertando o operador proativamente.
- **Arquitetura multi-clínica**: toda tabela e constraint única é isolada por `clinic_id`, não assume single-tenant.
- **Painel administrativo**: gestão de clínicas, visão ao vivo de conversas/agendamentos/fila, controle manual de status de cobrança e um CRM comercial (pipeline de lead até cliente pagante).
- **Ações de escrita auditadas**: ações administrativas sensíveis (pausar IA, reenfileirar mensagem falha, atualizar cobrança) são registradas em log, não silenciosas.

## Stack tecnológica

| Camada | Tecnologia |
|---|---|
| IA conversacional | APIs de LLM (modelos rápido/robusto em camadas) via agente orquestrado com chamada de ferramentas |
| Orquestração / motor de workflow | [n8n](https://n8n.io) (self-hosted, Docker) |
| Canal de mensagens | WhatsApp, via provedor terceiro de WhatsApp Cloud API |
| Transferência humana / caixa de suporte | [Chatwoot](https://www.chatwoot.com) (self-hosted) |
| Banco de dados / backend | [Supabase](https://supabase.com) (PostgreSQL, Row-Level Security, RPCs, Realtime) |
| Painel administrativo | React 18 + TypeScript + Vite + Tailwind CSS v4 + React Router, deploy na Vercel |
| Landing page | React 18 + TypeScript + Vite + Tailwind CSS v4, deploy na Vercel |
| Agenda | Google Calendar API (OAuth2) |
| Transcrição de voz | API de speech-to-text |

## Status do projeto

**Fase pré-escala / primeira clínica pagante.** Os fluxos principais — conversa, agendamento, transferência humana, lembretes e monitoramento — estão construídos e foram validados de ponta a ponta em ambiente de staging contra uma clínica-piloto real. A cobrança é deliberadamente manual nesta fase (ver [ADR-0008](docs/pt-BR/adrs/0008-cobranca-manual-antes-do-stripe.md)) — cobrança automatizada (Stripe, emissão de nota) fica para quando houver clínicas pagantes suficientes para justificar o custo de integração. Veja [`docs/pt-BR/roadmap.md`](docs/pt-BR/roadmap.md) para o que já foi entregue versus o que está planejado.

## Mapa da documentação

| Documento | O que cobre |
|---|---|
| [Requisitos de Sistema](docs/pt-BR/requisitos-de-sistema.md) | Requisitos funcionais, não funcionais, de segurança, observabilidade, escalabilidade, manutenção e integrações |
| [Regras de Negócio](docs/pt-BR/regras-de-negocio.md) | Regras de clínica, atendimento, transferência, horários, cobrança, qualificação de leads, mensagens e privacidade |
| [Arquitetura](docs/pt-BR/arquitetura.md) | Detalhamento de componentes, fluxo de dados, integrações, diagramas Mermaid |
| [Fluxos da Sofia IA](docs/pt-BR/fluxos-sofia-ia.md) | Fluxos de primeiro contato, qualificação, dúvidas, áudio, agendamento, objeções, transferência, fallback e follow-up |
| [Casos de Uso](docs/pt-BR/casos-de-uso.md) | Cenários ponta a ponta pelas personas de paciente, clínica e operador |
| [User Stories](docs/pt-BR/user-stories.md) | Histórias com critérios de aceite por persona |
| [Roadmap](docs/pt-BR/roadmap.md) | MVP entregue, melhorias de curto prazo, produto, técnicas, segurança/compliance, escalabilidade, comercial e futuro SaaS |
| [ADRs](docs/pt-BR/adrs/) | 8 registros de decisão técnica com contexto, alternativas e trade-offs |
| [Integrações](docs/pt-BR/integracoes.md) | Comparação entre API oficial do WhatsApp e provedor terceiro, além de Chatwoot, n8n, Supabase e a camada de IA |
| [Posicionamento de Portfólio](docs/pt-BR/posicionamento-portfolio.md) | Quais competências profissionais este projeto demonstra e onde ver evidência de cada uma |

Todo documento acima tem uma versão em inglês em [`docs/en/`](docs/en/).

## O que este projeto demonstra

Este estudo de caso busca mostrar, com evidência em vez de afirmação, a capacidade de:

- Levantar e estruturar requisitos para um produto operacional real, não um problema de exercício.
- Desenhar regras de negócio que se sustentam em casos de borda (corrida de consentimento, retries, isolamento multi-tenant, estouro de SLA).
- Arquitetar uma solução multi-sistema (IA, orquestração, banco de dados, mensageria, ferramenta de suporte, painel admin) e justificar por escrito cada escolha de tecnologia.
- Construir tanto a camada de automação (workflows) quanto a camada de software (apps React/TypeScript de admin e landing) — um perfil híbrido no-code/low-code + full-stack.
- Operar o que foi construído: monitoramento, alertas, runbooks de incidente e um rollout piloto real.

Veja [Posicionamento de Portfólio](docs/pt-BR/posicionamento-portfolio.md) para o detalhamento completo mapeado a competências específicas (análise de requisitos, análise de negócio, arquitetura de sistemas, pensamento de produto e desenvolvimento full-stack/híbrido).

## Contato

**Matheus Bittencourt** — [contact-email]

Este repositório é um artefato de portfólio. Ele exclui deliberadamente credenciais, detalhes de infraestrutura de produção e qualquer dado real de paciente ou prospect.
