# Requisitos de Sistema

> 🇺🇸 [Read in English](../en/system-requirements.md)

Legenda de status: **Implementado** (construído e validado ao menos em staging), **Parcial** (construído mas com uma lacuna conhecida), **Planejado** (no roadmap, ainda não construído), **`[a validar]`** (comportamento considerado verdadeiro, mas ainda não confirmado contra produção no momento da escrita).

## 1. Requisitos funcionais

| ID | Requisito | Status |
|---|---|---|
| FR-1 | O sistema deve receber mensagens de WhatsApp (texto e voz) no número configurado de uma clínica e roteá-las para a assistente de IA. | Implementado |
| FR-2 | O sistema deve transcrever mensagens de voz em texto antes de gerar a resposta da IA. | Implementado |
| FR-3 | O sistema deve manter contexto de conversa (histórico + memória resumida) por par paciente/clínica ao longo de múltiplas mensagens. | Implementado |
| FR-4 | A assistente de IA deve checar disponibilidade na agenda antes de confirmar um agendamento. | Implementado |
| FR-5 | A assistente de IA deve conseguir criar, e o sistema deve conseguir atualizar/cancelar, um agendamento, refletido tanto no banco quanto no Google Calendar. | Implementado |
| FR-6 | O sistema deve escalar uma conversa para um humano quando o paciente pedir explicitamente, o caso for sensível, a pergunta estiver fora de escopo, ou uma operação de agendamento falhar. | Implementado |
| FR-7 | Uma conversa transferida para humano deve incluir um resumo gerado por IA do que já foi conversado. | Implementado |
| FR-8 | A IA deve parar de responder a uma conversa no momento em que ela é transferida para um humano, e só deve retomar após o humano resolver (ou após uma política de reativação se aplicar). | Implementado |
| FR-9 | O sistema deve enviar lembretes automáticos de consulta antes do horário agendado e rastrear a confirmação do paciente. | Implementado |
| FR-10 | O sistema deve capturar e registrar o consentimento LGPD explícito de um paciente novo antes de gerar qualquer resposta de IA para ele. | Implementado |
| FR-11 | O painel admin deve permitir ao operador visualizar clínicas, conversas, fila de processamento e agendamentos de todas as clínicas. | Implementado |
| FR-12 | O painel admin deve permitir ao operador definir manualmente o status de cobrança de uma clínica (trial / ativo / atrasado / cancelado). | Implementado |
| FR-13 | O painel admin deve oferecer um CRM comercial (captura de lead, estágio do pipeline, notas) independente das visões de operação da clínica. | Implementado |
| FR-14 | A landing page pública deve oferecer um fluxo de qualificação de leads que grava um novo lead no CRM sem exigir que o visitante crie uma conta. | Implementado |
| FR-15 | O sistema deve suportar mais de uma clínica simultaneamente, cada uma com configuração, conversas e dados independentes. | Implementado |
| FR-16 | A recepção da clínica deve conseguir visualizar e responder conversas transferidas sem acessar o painel admin ou o n8n. | Implementado (via Chatwoot) |
| FR-17 | O sistema deve oferecer um portal de cobrança self-service para o cliente. | Planejado — ver [Roadmap](roadmap.md) |
| FR-18 | O sistema deve suportar cobrança recorrente automatizada (não apenas atualização manual de status). | Planejado — ver [ADR-0008](adrs/0008-cobranca-manual-antes-do-stripe.md) |

## 2. Requisitos não funcionais

| ID | Requisito | Status |
|---|---|---|
| NFR-1 | Uma mensagem do paciente deve receber resposta da IA dentro de uma janela de tempo que pareça conversacional (não lenta como bot de menu). | `[a validar: sem latência p95 formalmente medida ainda]` |
| NFR-2 | O sistema deve minimizar o custo de LLM por conversa, roteando perguntas simples para um modelo mais barato/rápido e só escalando complexidade para um modelo mais robusto. | Implementado |
| NFR-3 | O sistema deve degradar graciosamente quando uma dependência externa (Calendar, provedor de WhatsApp, Chatwoot) fica indisponível, em vez de perder a mensagem. | Parcial — o Calendar tem um caminho de fallback explícito; indisponibilidades de WhatsApp/Chatwoot dependem do mecanismo de retry da fila |
| NFR-4 | O painel admin deve refletir o estado operacional em tempo quase real sem exigir atualização manual. | Implementado — auto-refresh por polling (não é realtime via push em todas as visões) |
| NFR-5 | A memória de conversa deve ser limitada para que o tamanho do prompt (e portanto custo/latência) não cresça sem limite com o tamanho da conversa. | Implementado — memória em janela |
| NFR-6 | O sistema deve ser operável por uma única pessoa, sem uma equipe de operações dedicada. | Implementado — essa é uma restrição de design, não uma reflexão tardia (ver [ADR-0001](adrs/0001-orquestracao-com-n8n.md)) |

## 3. Requisitos de segurança

| ID | Requisito | Status |
|---|---|---|
| SEC-1 | Row-Level Security deve estar ativo em toda tabela que guarde dados de clínica ou paciente. | Implementado |
| SEC-2 | A chave anônima de banco de dados, exposta publicamente, não deve conseguir ler dados que ela mesma não inseriu (ex: leads do CRM). | Implementado — política de apenas-inserção na tabela `crm_leads` para o papel anônimo |
| SEC-3 | Operações privilegiadas (ex: mudança de status de cobrança) devem passar por um procedimento armazenado explícito e de escopo estreito, em vez de escrita direta na tabela a partir do código cliente. | Implementado |
| SEC-4 | Processos de backend confiáveis (n8n) podem usar um papel de banco de dados elevado, mas esse papel nunca deve ser exposto a nenhum frontend, arquivo exportado ou repositório público. | Implementado (disciplina operacional, não garantida por ferramenta — sinalizado como risco de processo manual) |
| SEC-5 | Segredos/credenciais de workflow não devem ser armazenados como variáveis de ambiente em texto plano, legíveis fora da plataforma de orquestração. | Implementado — usa-se o armazenamento de credenciais nativo do n8n; variáveis de ambiente não estão disponíveis nesse deploy (ver [Arquitetura § 7](arquitetura.md#7-fronteiras-de-segurança)) |
| SEC-6 | Credenciais de serviços terceiros devem ser rotacionadas periodicamente, não deixadas estáticas indefinidamente. | Planejado / em andamento — `[a validar: cadência de rotação ainda não formalizada]` |
| SEC-7 | Nenhum dado que identifique paciente deve aparecer em repositório público, arquivo de workflow exportado ou nesta documentação. | Implementado neste repositório por desenho (anonimização aplicada em todo o conteúdo) |

## 4. Requisitos de observabilidade

| ID | Requisito | Status |
|---|---|---|
| OBS-1 | O sistema deve registrar eventos estruturados de erro, aviso e informação, etiquetados por workflow e nó. | Implementado — tabela `logs` |
| OBS-2 | O sistema deve detectar e alertar sobre falhas de suas próprias dependências (banco de dados, provedor de WhatsApp, caixa de suporte humano) sem esperar a clínica reportar uma indisponibilidade. | Implementado — health checks agendados |
| OBS-3 | O sistema deve detectar padrões operacionais anômalos (ex: acúmulo de fila, volume de envio anormal) e alertar proativamente. | Implementado |
| OBS-4 | O sistema deve detectar e alertar sobre transferências humanas que estourem um SLA de tempo de resposta. | Implementado |
| OBS-5 | O alerta deve evitar notificações duplicadas/repetidas para o mesmo problema em andamento. | Implementado — lógica de cooldown por tipo de alerta |
| OBS-6 | O sistema deve ter um runbook de incidente documentado, cobrindo os modos de falha mais prováveis. | Implementado (runbook interno; não incluído neste estudo de caso público por confidencialidade) |
| OBS-7 | O sistema deveria expor métricas operacionais agregadas (mensagens/dia, agendamentos/dia, taxa de transferência) em um dashboard, não só logs brutos. | `[a validar: métricas existem de forma informal; uma visão de métricas dedicada está no roadmap]` |

## 5. Requisitos de escalabilidade

| ID | Requisito | Status |
|---|---|---|
| SCALE-1 | Toda tabela e constraint de unicidade deve ser isolada por clínica, sem assumir single-tenant. | Implementado |
| SCALE-2 | O processamento de mensagens deve ser assíncrono (em fila), em vez de tratado de forma síncrona dentro do webhook de entrada, para que uma chamada de IA lenta não bloqueie a ingestão. | Implementado |
| SCALE-3 | O sistema deve aplicar um rate limit por clínica para proteger a infraestrutura compartilhada de um pico de tráfego de uma única clínica. | Implementado |
| SCALE-4 | O sistema deve suportar crescimento horizontal no número de clínicas sem redesenho de schema. | Implementado na escala atual; `[a validar: ainda não testado sob carga além de uma clínica-piloto única]` |
| SCALE-5 | A cobrança deve conseguir evoluir de um processo totalmente manual para um automatizado conforme o número de clínicas cresce, sem uma reconstrução completa. | Planejado — deliberadamente adiado, ver [ADR-0008](adrs/0008-cobranca-manual-antes-do-stripe.md) |

## 6. Requisitos de manutenção

| ID | Requisito | Status |
|---|---|---|
| MAINT-1 | Staging e produção devem estar isolados para que mudanças em andamento nunca cheguem a pacientes reais. | Implementado — workflows de staging protegidos por allowlist |
| MAINT-2 | Mudanças de schema de banco de dados devem ser aplicadas via migrações versionadas, não edições ad hoc no banco em produção. | Parcial — a maioria das tabelas centrais é rastreada por migração; `crm_leads`/`crm_notes` são uma exceção conhecida (ver [Arquitetura § 9](arquitetura.md#9-débito-arquitetural-conhecido)) |
| MAINT-3 | As definições de workflow devem ser exportáveis/backupáveis fora da ferramenta de orquestração. | Implementado — exports JSON de workflow são versionados |
| MAINT-4 | O código e os workflows devem documentar quais credenciais/dependências cada componente exige. | Implementado (mapa de dependências interno; não publicado aqui por confidencialidade) |
| MAINT-5 | Os deploys deveriam ser automatizáveis (CI/CD), não exclusivamente manuais. | Planejado — hoje é manual |

## 7. Requisitos de integração

| ID | Requisito | Status |
|---|---|---|
| INT-1 | O sistema deve enviar e receber mensagens de WhatsApp por meio de um provedor compatível com Cloud API. | Implementado — ver [Integrações](integracoes.md) |
| INT-2 | O sistema deve se integrar a uma caixa de suporte humano que a equipe da clínica consiga usar sem treinamento técnico. | Implementado — Chatwoot |
| INT-3 | O sistema deve se integrar ao Google Calendar para checagem de disponibilidade e criação de eventos. | Implementado |
| INT-4 | O sistema deve se integrar a um provedor de LLM com suporte a chamada de ferramentas (function calling), não apenas completions de texto simples. | Implementado |
| INT-5 | O sistema deve se integrar a um provedor de speech-to-text para suporte a mensagens de voz. | Implementado |
| INT-6 | A camada de banco de dados deve expor notificações de mudança em tempo real para o painel admin consumir, além de consultas sob demanda. | Parcial — Realtime está ativo nas tabelas centrais; o painel admin hoje depende principalmente de polling |
| INT-7 | O sistema deve suportar uma futura migração para a API oficial do WhatsApp Business sem uma reescrita completa de arquitetura. | Objetivo de design — ver [ADR-0004](adrs/0004-uazapi-vs-api-oficial-whatsapp.md) |
