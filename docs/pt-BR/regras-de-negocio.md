# Regras de Negócio

> 🇺🇸 [Read in English](../en/business-rules.md)

Estas regras foram extraídas das constraints reais do schema, da lógica das RPCs e da documentação interna de processo — são as restrições que o sistema de fato aplica ou que o operador de fato segue, não política aspiracional. Tudo que não é aplicado em código está identificado como tal.

## 1. Regras de clínicas

- Cada clínica é um tenant independente: número de WhatsApp próprio, horário de funcionamento, prompt de IA, plano, mapeamento de agenda e rate limit próprios. Nenhum dado é compartilhado entre `clinic_id`.
- Uma clínica pode ser desativada (`active = false`) sem apagar seus dados — a desativação para a IA e os lembretes, mas preserva o histórico pelo período de retenção descrito abaixo.
- A IA de uma clínica pode ser pausada no nível da clínica (todas as conversas) ou no nível de uma conversa individual (um único paciente) — são dois controles diferentes para duas situações diferentes (ex: "a clínica está fechada num feriado" vs. "esse paciente específico precisa de um humano").
- Uma clínica tem um rate limit configurável (mensagens por minuto), aplicado antes do enfileiramento — isso protege a infraestrutura compartilhada caso o tráfego de uma clínica dispare de forma anormal (ex: uma campanha de disparo que a própria clínica enviou).

## 2. Regras de atendimento / conversa

- Uma conversa é identificada de forma única por `(telefone, clinic_id)` — o mesmo número de telefone falando com duas clínicas diferentes gera duas conversas independentes, com contexto independente, e não uma conversa vazando entre tenants.
- Nenhuma resposta de IA é gerada para uma conversa com consentimento LGPD não resolvido — isso é uma trava obrigatória no Worker de processamento, avaliada antes de toda chamada à IA, não uma instrução de prompt que o modelo poderia ignorar.
- Toda mensagem recebida é deduplicada pelo ID de mensagem atribuído pelo provedor antes de ser processada — um retry de entrega do WhatsApp nunca pode gerar duas respostas de IA para a mesma mensagem.
- A Sofia não pode: dar diagnóstico clínico, prometer resultado específico de procedimento, cotar preço não configurado explicitamente para aquela clínica, ou continuar respondendo depois que a conversa está em atendimento humano.
- O contexto da conversa é limitado (memória em janela + resumo periódico), não armazenado e repetido por completo para sempre — isso é tanto um controle de custo quanto uma regra sobre o que "contexto" significa operacionalmente.

## 3. Regras de transferência humana (handoff)

- Uma conversa deve ser transferida para um humano quando qualquer um destes ocorrer: o paciente pede explicitamente um humano, o conteúdo da mensagem é sinalizado como sensível, a pergunta está fora do escopo configurado da IA, ou uma operação automatizada de agendamento falha.
- No momento em que a transferência é disparada, a IA é pausada imediatamente naquela conversa — não existe uma janela em que tanto a IA quanto um humano possam responder ao mesmo paciente.
- Toda transferência deve chegar à caixa de entrada do humano (Chatwoot) com um resumo gerado por IA do que já foi conversado — nunca se espera que um humano role o histórico bruto do chat para pegar contexto.
- Uma conversa transferida é etiquetada por clínica, então a equipe da clínica que recebe só vê os próprios pacientes mesmo dentro de uma ferramenta de suporte compartilhada.
- A IA volta a atuar automaticamente assim que um humano marca a conversa como resolvida — a transferência é um loop de volta para a automação por padrão, não uma entrega permanente.
- Uma conversa abandonada no meio de uma transferência, além de um limite de inatividade definido, é elegível para reativação automática: a IA envia um follow-up gerado e retoma a conversa, em vez de deixar o paciente permanentemente parado numa fila humana sem resposta.

## 4. Regras de agendamento / horários

- A disponibilidade é sempre checada contra a agenda real da clínica antes de um agendamento ser confirmado — a IA não agenda de forma especulativa para corrigir depois.
- Uma falha de agendamento (ex: erro na API do calendário) é roteada para transferência humana, em vez de falhar silenciosamente ou responder ao paciente "tente mais tarde" sem nenhum follow-up.
- Lembretes de consulta são enviados em um cronograma fixo relativo à consulta (uma janela do dia anterior e uma do mesmo dia), e o envio/confirmação de cada lembrete é rastreado por agendamento, não presumido.
- O status do agendamento segue um ciclo de vida definido (pendente → agendado → confirmado → concluído, ou → cancelado), e cancelamentos exigem que um motivo seja registrado.

## 5. Regras de cobrança / trial

- Uma clínica nova começa em status de cobrança `trial`, com um período de trial definido; as transições de status (`trial → ativo → atrasado → cancelado`) são decisão do operador, feitas através de uma única ação administrativa restrita — não são automáticas com base em webhooks de pagamento, porque a cobrança hoje é um processo manual por decisão de design (ver [ADR-0008](adrs/0008-cobranca-manual-antes-do-stripe.md)).
- Somente a conta do operador pode executar uma mudança de status de cobrança — isso é garantido na camada de banco de dados (uma RPC `SECURITY DEFINER` verifica quem está chamando), não apenas escondido na interface.
- Cobrança em atraso dispara um aviso visível dentro da própria visão da clínica no painel, e o cancelamento segue um processo definido: sem multa, sem cobrança proporcional, dados retidos por um período mínimo após o cancelamento em vez de apagados imediatamente.
- Cobrança recorrente automatizada, emissão de nota fiscal e um portal de cobrança self-service estão explicitamente fora de escopo até haver clínicas pagantes suficientes para justificar a integração — uma decisão deliberada de sequenciamento "manual até doer", não um esquecimento.

## 6. Regras de qualificação de leads

- Um lead entra no CRM por uma de três origens: o formulário de lead da landing page pública, o fluxo interativo de qualificação "diagnóstico", ou entrada manual pelo operador — cada um é marcado com sua origem.
- Um lead avança por uma sequência definida de estágios de pipeline (novo → qualificado → contatado → demo marcada → piloto ativo → cliente, ou → perdido com um motivo registrado) — mudanças de estágio não são atualizações de status em texto livre.
- O formulário público de lead e o fluxo de qualificação só conseguem criar um lead, nunca ler o pipeline de volta — o tráfego público não pode enumerar ou raspar o pipeline comercial.
- Mover um lead de "qualificado" para "demo marcada" através do fluxo público de diagnóstico acontece via um procedimento armazenado restrito, não uma escrita direta na tabela, para manter a superfície de escrita pública o mais estreita possível mesmo que o fluxo precise de algum acesso de escrita.
- Critérios de cliente ideal usados para priorizar a qualificação outbound (não é uma restrição rígida do sistema, é uma regra de processo): 1 a 5 profissionais, já agenda via WhatsApp, usa Google Calendar/Agenda, e o dono é quem decide diretamente.

## 7. Regras de mensagens e limites

- Cada clínica tem um rate limit por minuto aplicado no processamento de saída, para proteger a infraestrutura compartilhada — tráfego acima do limite entra em fila em vez de disparar imediatamente.
- Falhas repetidas de processamento da mesma mensagem são reprocessadas com backoff exponencial até um teto fixo de tentativas, após o qual o item vai para um estado de dead-letter e é sinalizado ao operador em vez de tentar indefinidamente.
- Volume de envio diário anormalmente alto para uma clínica dispara um alerta operacional — isso é tratado como um sinal que merece investigação (configuração errada, abuso ou um loop descontrolado), não apenas folga de capacidade.
- Mensagens de voz são sempre transcritas antes de serem tratadas como entrada de conversa — a IA nunca processa áudio bruto diretamente como parte do seu raciocínio.

## 8. Regras de segurança e privacidade

- Toda mensagem de paciente exige consentimento registrado antes que a IA possa responder; os registros de consentimento são imutáveis e nunca são apagados, já que servem como prova legal de consentimento, não apenas estado operacional.
- Logs operacionais são retidos por um período limitado (curto por padrão), exceto quando explicitamente marcados como registro de auditoria.
- Histórico de conversa e memória de trabalho são apagados após um período definido de inatividade do paciente; registros de agendamento são anonimizados (não apagados por completo) após uma janela de retenção mais longa, equilibrando necessidades legais/de registro com minimização de dados.
- Nenhuma aplicação frontend (painel admin ou landing page) tem privilégios elevados de banco de dados — só a camada de orquestração de backend confiável tem, e apenas para as operações que exigem isso.
- Nenhuma credencial de produção, número de telefone de paciente ou identidade real de clínica pode aparecer em um arquivo de workflow exportado, nesta documentação, ou em qualquer repositório público — essa regra moldou a forma como este próprio estudo de caso foi escrito (ver a nota de anonimização no [README](../../README.pt-BR.md)).
