# Casos de Uso

> 🇺🇸 [Read in English](../en/use-cases.md)

Cada caso de uso segue: **Ator → Gatilho → Pré-condições → Fluxo principal → Resultado → Regras relacionadas.**

## UC-1 — Paciente novo faz o primeiro contato

- **Ator**: Paciente (número de telefone novo para essa clínica)
- **Gatilho**: Paciente envia qualquer mensagem de WhatsApp para o número da clínica.
- **Pré-condições**: Clínica está ativa e a IA não está pausada no nível da clínica.
- **Fluxo principal**:
  1. O Receptor recebe o webhook, resolve a clínica, aplica rate limit, deduplica pelo ID da mensagem.
  2. O sistema detecta que não existe registro de consentimento para esse par telefone/clínica.
  3. O sistema envia um pedido de consentimento LGPD em vez de rotear para a Sofia.
  4. Paciente aceita → consentimento registrado → a conversa segue normalmente a partir da próxima mensagem.
- **Resultado**: Existe um registro de conversa em conformidade antes de qualquer conteúdo gerado por IA ser enviado a um paciente novo.
- **Regras relacionadas**: [Regras de Negócio § 2, § 8](regras-de-negocio.md); [Fluxos da Sofia IA § 1](fluxos-sofia-ia.md#1-fluxo-de-primeira-mensagem).

## UC-2 — Lead pede preço

- **Ator**: Paciente / potencial paciente
- **Gatilho**: Uma mensagem perguntando o preço de um serviço.
- **Pré-condições**: Consentimento já registrado; conversa em estado de IA ativa.
- **Fluxo principal**:
  1. A Sofia checa se o preço do serviço perguntado está configurado para essa clínica.
  2. Se estiver configurado, a Sofia responde com o preço/faixa configurada.
  3. Se não estiver configurado, a Sofia não inventa um número — ou diz que o preço depende de avaliação, ou escala se o paciente insistir num valor exato que a Sofia não pode fornecer.
- **Resultado**: O paciente nunca recebe um preço que a clínica não autorizou.
- **Regras relacionadas**: [Regras de Negócio § 2](regras-de-negocio.md#2-regras-de-atendimento--conversa); [Fluxos da Sofia IA § 6](fluxos-sofia-ia.md#6-fluxo-de-tratamento-de-objeções).

## UC-3 — Lead envia uma mensagem de áudio

- **Ator**: Paciente
- **Gatilho**: Paciente envia uma mensagem de áudio em vez de texto.
- **Pré-condições**: Consentimento já resolvido.
- **Fluxo principal**:
  1. O Receptor detecta que a mensagem é áudio e baixa o arquivo.
  2. O áudio é transcrito para texto via a integração de speech-to-text.
  3. A transcrição é tratada como o corpo da mensagem e entra no fluxo normal de processamento.
  4. A Sofia responde como se o paciente tivesse digitado a mensagem.
- **Resultado**: Pacientes de voz e de texto recebem um atendimento funcionalmente idêntico.
- **Regras relacionadas**: [Fluxos da Sofia IA § 4](fluxos-sofia-ia.md#4-fluxo-de-mensagem-de-voz-áudio).

## UC-4 — Lead quer agendar uma avaliação

- **Ator**: Paciente
- **Gatilho**: Paciente pede para agendar uma consulta/avaliação.
- **Pré-condições**: Consentimento resolvido; agenda da clínica configurada.
- **Fluxo principal**:
  1. A Sofia chama a tool de disponibilidade para o serviço/profissional/faixa de data pedidos.
  2. Os horários disponíveis são apresentados ao paciente.
  3. O paciente confirma um horário.
  4. A Sofia chama a tool de agendamento: evento criado no Calendar, linha gravada em `appointments`, confirmação enviada.
  5. Caminho alternativo: se o agendamento falhar, a conversa é transferida para um humano em vez de falhar silenciosamente.
- **Resultado**: Existe um agendamento real, respaldado pela agenda, ou o paciente está na fila de um humano com contexto completo — nunca um beco sem saída.
- **Regras relacionadas**: [Fluxos da Sofia IA § 5](fluxos-sofia-ia.md#5-fluxo-de-agendamento); [Regras de Negócio § 4](regras-de-negocio.md#4-regras-de-agendamento--horários).

## UC-5 — Lead pede atendimento humano

- **Ator**: Paciente
- **Gatilho**: Paciente pede explicitamente para falar com uma pessoa (ou o sistema detecta um caso sensível/fora de escopo).
- **Pré-condições**: Nenhuma além de uma conversa ativa.
- **Fluxo principal**:
  1. O workflow de Transferência dispara: briefing por IA gerado, IA pausada nessa conversa.
  2. Contato/conversa no Chatwoot encontrado ou criado, briefing publicado, label da clínica aplicada.
  3. Equipe interna notificada; paciente informado de que será atendido por uma pessoa.
  4. A equipe da clínica assume a conversa no Chatwoot já com todo o contexto disponível.
- **Resultado**: Nenhuma transferência a frio — um humano sempre começa com contexto, nunca com um chat em branco.
- **Regras relacionadas**: [Regras de Negócio § 3](regras-de-negocio.md#3-regras-de-transferência-humana-handoff); [Fluxos da Sofia IA § 7](fluxos-sofia-ia.md#7-fluxo-de-transferência-para-humano).

## UC-6 — Clínica acompanha suas próprias conversas

- **Ator**: Recepção da clínica
- **Gatilho**: A equipe entra no Chatwoot para checar as conversas com pacientes.
- **Pré-condições**: A equipe tem acesso ao Chatwoot restrito à label da própria clínica.
- **Fluxo principal**:
  1. A equipe vê apenas conversas etiquetadas para sua clínica, incluindo histórico atendido pela IA e conversas transferidas.
  2. A equipe pode assumir uma conversa, responder, e marcar como resolvida (o que reativa a IA automaticamente).
- **Resultado**: A equipe da clínica ganha visibilidade operacional sem precisar de acesso ao painel admin ou ao banco de dados.
- **Regras relacionadas**: [Arquitetura § 3.4](arquitetura.md#34-transferência-humana-chatwoot).

## UC-7 — Admin altera o status de cobrança de uma clínica

- **Ator**: Operador do ZapCare
- **Gatilho**: O trial de uma clínica termina, um pagamento é confirmado, ou um pagamento está em atraso.
- **Pré-condições**: Operador autenticado no painel admin.
- **Fluxo principal**:
  1. Operador abre a linha da clínica na visão de Clínicas.
  2. Operador seleciona o novo status de cobrança (ex: trial → ativo).
  3. O painel chama a RPC restrita de cobrança, que verifica se quem está chamando é a conta autorizada do operador antes de aplicar a mudança.
  4. A ação é registrada em log.
  5. Se o novo status for `atrasado`, a própria visão da clínica no painel exibe um banner de aviso.
- **Resultado**: Mudanças de estado de cobrança são deliberadas, atribuíveis e restritas a um único caminho autorizado.
- **Regras relacionadas**: [Regras de Negócio § 5](regras-de-negocio.md#5-regras-de-cobrança--trial); [Arquitetura § 7](arquitetura.md#7-fronteiras-de-segurança).

## UC-8 — Operador acompanha logs e falhas

- **Ator**: Operador do ZapCare
- **Gatilho**: Um health check agendado, o SLA monitor, ou um alerta do retry scheduler dispara — ou o operador checa proativamente as visões de Fila/Monitoramento.
- **Pré-condições**: Nenhuma.
- **Fluxo principal**:
  1. Operador recebe um alerta (ex: acúmulo de fila, integração inacessível, estouro de SLA) pelo canal de alerta.
  2. Operador abre a visão de Monitoramento/Fila do painel admin para ver o(s) item(ns) específico(s) com falha e seu contexto de erro registrado.
  3. Operador resolve o problema de base (ex: reconecta uma integração) e, se necessário, reenfileira manualmente uma mensagem falha pelo painel — uma ação que também é registrada em log.
- **Resultado**: Falhas ficam visíveis e acionáveis em minutos, não descobertas dias depois por uma reclamação de clínica.
- **Regras relacionadas**: [Requisitos de Sistema § 4 (Observabilidade)](requisitos-de-sistema.md#4-requisitos-de-observabilidade); [Arquitetura § 6](arquitetura.md#6-mecanismos-de-confiabilidade).

## UC-9 — Sistema detecta um erro de integração

- **Ator**: Sistema (workflow Health Check)
- **Gatilho**: Execução agendada (a cada 5 minutos).
- **Pré-condições**: Nenhuma — isso roda incondicionalmente por agendamento.
- **Fluxo principal**:
  1. O Health Check testa o banco de dados, o provedor de WhatsApp e o Chatwoot.
  2. Também checa acúmulo de fila e volume de envio diário anormal.
  3. Se algum check falhar ou cruzar um limite, e o alerta correspondente não estiver em cooldown, um alerta é enviado ao operador.
  4. Cada tipo de falha tem seu próprio cooldown, para que uma indisponibilidade persistente gere um único fluxo de alerta, não uma enxurrada.
- **Resultado**: O sistema reporta sua própria degradação antes que um paciente ou clínica perceba — monitoramento proativo, não reativo.
- **Regras relacionadas**: [Arquitetura § 6](arquitetura.md#6-mecanismos-de-confiabilidade); [Requisitos de Sistema § 4](requisitos-de-sistema.md#4-requisitos-de-observabilidade).
