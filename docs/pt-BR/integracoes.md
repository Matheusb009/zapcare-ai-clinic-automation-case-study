# Integrações

> 🇺🇸 [Read in English](../en/integrations.md)

Este documento compara os serviços externos que o ZapCare integra, por que cada um foi escolhido nesta fase, e quais são os trade-offs. Para a narrativa de decisão por trás das duas escolhas mais debatidas (n8n e UazAPI), ver [ADR-0001](adrs/0001-orquestracao-com-n8n.md) e [ADR-0004](adrs/0004-uazapi-vs-api-oficial-whatsapp.md).

## 1. Mensageria WhatsApp: API oficial do WhatsApp Business vs. UazAPI

| | API oficial do WhatsApp Business (via um Business Solution Provider) | UazAPI (usada hoje) |
|---|---|---|
| Complexidade de setup | Maior — exige verificação de negócio na Meta, número registrado e, tipicamente, contrato com um BSP | Menor — conecta um número de WhatsApp já existente via um gateway hospedado, quase plug-and-play |
| Custo em baixo volume | Geralmente tem um piso de custo fixo/mínimo mais alto, feito para escala | Custo de entrada menor, cabe num piloto de uma única clínica |
| Tempo até a primeira integração funcionando | Dias a semanas (verificação de negócio, registro de número) | Horas |
| Risco de compliance / ToS | Totalmente dentro do caminho de integração suportado pelo WhatsApp | Opera fora do caminho de API oficialmente sancionado pelo WhatsApp — carrega um risco real de restrição do número se o padrão de mensagens parecer automatizado/spam `[risco, ainda não materializado]` |
| Exigência de templates de mensagem | Rígida (templates pré-aprovados para mensagens iniciadas pela empresa fora da janela de 24h) | Sem restrição no nível do gateway, mas a mesma lógica de janela de 24h do protocolo do WhatsApp ainda se aplica |
| Melhor encaixe | Um produto em escala, com muitas clínicas e receita para sustentar custos de BSP | Validar product-market fit com um número pequeno de clínicas-piloto antes de assumir infraestrutura mais pesada |

**Por que UazAPI agora**: a prioridade nesta fase é validar se as clínicas de fato tiram valor da assistente — não construir para uma escala que ainda não existe. A UazAPI permitiu colocar o primeiro piloto no ar em horas em vez de semanas. O trade-off explicitamente aceito é o risco de compliance/ToS, acompanhado como um item real a ser revisitado antes de escalar de forma significativa o número de clínicas (ver [ADR-0004](adrs/0004-uazapi-vs-api-oficial-whatsapp.md)).

**Próximo passo**: migrar para a API oficial quando o número de clínicas e a receita justificarem o custo de BSP e o setup mais pesado — o sistema foi mantido deliberadamente desacoplado (todo I/O de WhatsApp passa pelos workflows Receptor/Worker, não espalhado pelo código) exatamente para que essa migração seja uma troca de provedor, não uma reescrita.

## 2. Chatwoot (transferência humana / caixa de suporte)

- **Vantagens**: open-source e self-hostable (sem custo por assento de SaaS nessa escala), feito exatamente para esse caso de uso (caixa de suporte multicanal com labels, atribuição e uma API de webhook para eventos de resolução), e interface familiar para a equipe não-técnica da clínica.
- **Limitações**: self-hosting significa que o ZapCare é dono do próprio uptime e patching, não um SLA de fornecedor. `[a validar: histórico de uptime atual]`.
- **Riscos**: uma indisponibilidade do Chatwoot remove completamente o caminho de transferência humana — é por isso que o Health Check monitora especificamente a disponibilidade do Chatwoot (ver [Arquitetura § 6](arquitetura.md#6-mecanismos-de-confiabilidade)).
- **Próximos passos**: nenhum planejado nesta fase — o Chatwoot não tem sido um gargalo.

## 3. n8n (orquestração)

- **Vantagens**: workflows visuais tornam a lógica do sistema auditável sem precisar ler código-fonte linha a linha — relevante tanto para a manutenibilidade de um operador solo quanto para explicar o sistema a um stakeholder não-técnico. Agendamento nativo, tratamento de erro via Error Trigger e gerenciamento de credenciais já vieram prontos, sem precisar construir essa infraestrutura do zero.
- **Limitações**: lógica de workflow é mais difícil de testar unitariamente do que código puro, e diffs em JSON exportado não são amigáveis para leitura humana — um custo real documentado em [Arquitetura § 9](arquitetura.md#9-débito-arquitetural-conhecido) e no [ADR-0001](adrs/0001-orquestracao-com-n8n.md).
- **Riscos**: a limitação de variáveis de ambiente deste deploy Docker específico (`$env.*` indisponível) força as credenciais para dentro do armazenamento próprio do n8n — aceitável, mas uma restrição que vale conhecer antes de presumir que uma configuração 12-factor padrão simplesmente funcionaria.
- **Próximos passos**: conforme o número e a complexidade de workflows crescer, avaliar consolidar os workflows duplicados STG/prod em versões únicas parametrizadas.

## 4. Supabase (banco de dados / backend)

- **Vantagens**: PostgreSQL gerenciado com Row-Level Security, subscriptions Realtime e acesso REST/RPC autogerado significaram que o painel admin e a landing page puderam falar diretamente com o banco, sem precisar construir e operar uma API de backend separada — um ganho real de velocidade para um produto construído por uma única pessoa.
- **Limitações**: a corretude das políticas de RLS é todo o modelo de segurança para escritas públicas (a chave anônima da landing page) — isso concentra o risco na revisão de política em vez de distribuí-lo por uma camada de API com validação própria.
- **Riscos**: duas das tabelas mais novas (`crm_leads`, `crm_notes`) existem só no banco em produção, não em migrações versionadas — uma lacuna conhecida (ver [Arquitetura § 9](arquitetura.md#9-débito-arquitetural-conhecido)).
- **Próximos passos**: preencher as migrações faltantes para as tabelas de CRM; formalizar um único fluxo de migração (hoje existem duas pastas de migração separadas).

## 5. A camada de IA (LLM + speech-to-text)

- **Vantagens**: suporte a chamada de ferramentas (function calling) era um requisito obrigatório — é isso que faz o fluxo de agendamento da Sofia gravar num calendário real em vez de alucinar um texto de confirmação. O roteamento em camadas entre um modelo rápido/barato e um modelo mais robusto mantém o custo médio por conversa baixo, ainda escalando perguntas genuinamente complexas para um modelo mais capaz.
- **Limitações**: o comportamento do modelo (tom, limites de recusa, confiabilidade de chamada de ferramenta) muda entre versões/provedores — prompts e proteções precisam de revalidação periódica, não a presunção de "configurar uma vez e esquecer".
- **Riscos**: uma indisponibilidade do provedor de LLM trava todas as conversas simultaneamente, em todas as clínicas — hoje não existe um provedor de fallback. `[a validar: aceitável na escala atual, vale revisitar antes que deixe de ser]`.
- **Próximos passos**: formalizar um processo leve de avaliação (eval) para mudanças de prompt/modelo antes de chegarem à produção, já que hoje não existe uma suíte de testes automatizada para qualidade conversacional.

## 6. Google Calendar

- **Vantagens**: as clínicas já usam — fricção de migração zero para a clínica, e é a fonte de verdade contra a qual a disponibilidade é checada, evitando um segundo sistema de agenda que a clínica precisaria manter sincronizado manualmente.
- **Limitações**: credenciais OAuth2 podem expirar/exigir reconexão, o que já causou incidentes reais (ver o runbook interno, não publicado aqui). Uma falha de agendamento por causa do Calendar tem um fallback definido (transferência humana), mas uma indisponibilidade do Calendar ainda degrada a experiência de agendamento.
- **Próximos passos**: `[a validar: se uma melhoria de redundância ou alerta para o calendário está planejada]`.
