# Roadmap

> 🇺🇸 [Read in English](../en/roadmap.md)

Este roadmap reflete o backlog real, acompanhado internamente — agrupado por frente em vez de por data, já que as prioridades de um produto operado por uma única pessoa mudam conforme a necessidade da clínica-piloto atual. Os itens são descritos como entregues ou não; datas específicas são intencionalmente omitidas onde ainda não há compromisso firmado.

## MVP — entregue

- Fluxo conversacional ponta a ponta: gate de consentimento → Sofia (IA em camadas) → agendamento ou transferência → confirmação.
- Suporte a mensagem de voz (transcrição → texto → resposta de IA).
- Transferência humana para o Chatwoot com briefing gerado por IA, e reativação automática da IA na resolução.
- Agendamento e cancelamento de consulta, sincronizado com Google Calendar.
- Lembretes automáticos de consulta com rastreamento de confirmação.
- Fila de mensagens assíncrona com retry, backoff exponencial e tratamento de dead-letter.
- Health checks agendados, monitoramento de SLA para transferências, e reativação de conversas abandonadas.
- Isolamento de dados multi-clínica no nível de schema.
- Painel admin: gestão de clínicas, visões de conversa/agendamento/fila, controle manual de cobrança, CRM comercial.
- Landing page pública com fluxo interativo de qualificação de leads alimentando o CRM.
- Processo de rollout em etapas: ambiente de staging com gate de allowlist, validado contra uma clínica-piloto real antes de um rollout mais amplo.

## Melhorias de curto prazo

- Resolver os achados remanescentes de validação da última rodada de teste ponta a ponta antes de trazer mais clínicas. `[a validar: status atual da lista de pendências]`
- Consolidar os dois fluxos separados de migração SQL num histórico cronológico único.
- Preencher migrações versionadas para as tabelas de CRM (`crm_leads`, `crm_notes`), hoje geridas apenas no banco em produção.
- Rodada formal de rotação de credenciais para tokens de serviços terceiros (provedor de WhatsApp, Chatwoot), como medida preventiva e não resposta a incidente.

## Melhorias de produto

- Fluxo de onboarding de clínica que não exija o operador configurar cada campo manualmente.
- Fluxos de confirmação automática de agendamento além do padrão atual de lembrete/confirmação.
- Dashboard de métricas operacionais agregadas (mensagens/dia, agendamentos/dia, taxa de transferência) em vez de depender de visões brutas de log/fila.
- Funcionalidades de CRM de paciente/lead mais robustas conforme o número de clínicas cresce (hoje dimensionado para um grupo piloto pequeno).

## Melhorias técnicas

- CI/CD para o painel admin e a landing page (hoje deploys manuais via `vercel --prod`).
- Consolidar a duplicação de workflows staging/produção (Receptor, Worker, Sofia) em workflows únicos parametrizados onde for prático.
- Um processo leve de avaliação para mudanças de prompt/modelo de IA antes de chegarem à produção.
- Atualizações em tempo real via push em mais visões do painel admin, reduzindo a dependência de polling.

## Segurança e compliance

- Formalizar a cadência de rotação de credenciais de terceiros (hoje é ad hoc/preventiva, não agendada).
- Revisão jurídica dos períodos atuais de retenção de dados por LGPD (logs, histórico de conversa, janelas de anonimização de agendamento).
- Avaliar necessidade de acordo de processamento de dados (DPA) com provedores de IA/infraestrutura.
- Estender a cobertura de Row-Level Security para qualquer tabela remanescente que hoje dependa de bypass via `service_role` em vez de proteção em nível de política. `[a validar: lacunas de cobertura atuais]`

## Escalabilidade

- Testar a arquitetura sob carga além do padrão de tráfego de uma única clínica-piloto, antes de assumir uma onda maior de onboarding.
- Revisitar a decisão UazAPI-vs-API-oficial-do-WhatsApp quando o número de clínicas e a receita justificarem a migração (ver [ADR-0004](adrs/0004-uazapi-vs-api-oficial-whatsapp.md)).
- Desenhar o caminho de transição de cobrança manual para cobrança automatizada (Stripe ou equivalente) antes de realmente precisar dele (ver [ADR-0008](adrs/0008-cobranca-manual-antes-do-stripe.md)).

## Comercial / prospecção

- Sistematizar a qualificação outbound contra o perfil de cliente ideal definido (1 a 5 profissionais, já agenda via WhatsApp hoje, usuário de Google Calendar/Agenda, decisão liderada pelo dono).
- Transformar os estágios do pipeline do CRM comercial em playbooks repetíveis (roteiro de demo, tratamento de objeção, processo de conversão piloto-para-pago).
- Rastrear a taxa de conversão piloto-para-cliente-pagante como a métrica central de validação comercial antes de escalar volume de outbound.

## Direção futura de SaaS

- Portal de cobrança self-service para clínicas (explicitamente adiado — ver [ADR-0008](adrs/0008-cobranca-manual-antes-do-stripe.md)).
- Suporte completo a múltiplos médicos/profissionais além do mapeamento de agenda por clínica atual.
- Funcionalidade básica de prontuário, se validada como necessidade real de clínica e não como scope creep.
- Campanhas formais de reativação de paciente/lead como funcionalidade de produto independente (hoje, a reativação existe só para transferências humanas abandonadas, não para win-back de marketing).
- Um painel admin com permissões e múltiplos usuários (hoje, efetivamente de operador único), com self-service de configuração por clínica.
