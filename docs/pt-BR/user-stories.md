# User Stories

> 🇺🇸 [Read in English](../en/user-stories.md)

Formato: **Como [persona], eu quero [objetivo], para que [benefício].** Cada história inclui critérios de aceite.

## Dono da clínica

**US-1**: Como dono da clínica, eu quero que as mensagens dos pacientes sejam respondidas mesmo fora do horário comercial, para que eu pare de perder leads por demora na resposta.
- Critérios de aceite:
  - Dado que um paciente manda mensagem para o WhatsApp da clínica a qualquer hora, quando o consentimento já está resolvido, então a Sofia responde sem esperar um funcionário estar online.
  - Dado que a IA não está pausada para a clínica, as respostas acontecem sem intervenção manual.

**US-2**: Como dono da clínica, eu quero que minha agenda seja a única fonte de verdade sobre disponibilidade, para que a IA nunca agende em duplicidade ou ofereça um horário que não existe.
- Critérios de aceite:
  - Dado que um paciente pede um horário, quando a Sofia checa disponibilidade, então o resultado reflete o estado real da agenda conectada no momento da consulta.
  - Dado que um agendamento é confirmado, existe um evento correspondente na agenda.

**US-3**: Como dono da clínica, eu quero saber quando minha mensalidade está vencendo ou vencida, para que eu não seja pego de surpresa por uma interrupção de serviço.
- Critérios de aceite:
  - Dado que meu status de cobrança muda para atrasado, quando eu abro o painel admin, então vejo um aviso visível com um caminho para resolver.
  - Dado que estou dentro do período de trial, vejo quantos dias restam.

## Recepcionista

**US-4**: Como recepcionista, eu quero receber uma conversa com contexto completo quando ela é transferida para mim, para que eu não precise pedir ao paciente para repetir tudo.
- Critérios de aceite:
  - Dado que uma conversa é escalada, quando ela aparece no Chatwoot, então inclui um resumo gerado por IA do que já aconteceu.
  - Dado que eu abro a conversa, as mensagens originais do paciente continuam visíveis acima do resumo.

**US-5**: Como recepcionista, eu quero que a IA fique fora de uma conversa depois que eu assumo, para que o paciente não receba duas respostas conflitantes.
- Critérios de aceite:
  - Dado que uma conversa está em atendimento humano, quando o paciente manda outra mensagem, então a Sofia não gera resposta.
  - Dado que eu resolvo a conversa no Chatwoot, a IA volta a atuar automaticamente nas próximas mensagens.

**US-6**: Como recepcionista, eu quero ver apenas as conversas da minha própria clínica, para que eu nunca veja dados de paciente de outra clínica.
- Critérios de aceite:
  - Dado que estou logada no Chatwoot com o acesso da minha clínica, quando eu navego pelas conversas, então só aparecem conversas etiquetadas para minha clínica.

## Paciente / lead

**US-7**: Como paciente, eu quero mandar uma mensagem de voz em vez de digitar, para que eu consiga entrar em contato da forma mais fácil para mim.
- Critérios de aceite:
  - Dado que eu envio uma mensagem de áudio, quando o sistema processa, então recebo uma resposta como se eu tivesse digitado o mesmo conteúdo.

**US-8**: Como paciente, eu quero ser perguntado sobre consentimento antes de uma IA começar a me mandar mensagens, para que eu entenda e controle como meus dados são usados.
- Critérios de aceite:
  - Dado que eu sou um contato novo para uma clínica, quando eu mando minha primeira mensagem, então recebo um pedido de consentimento antes de qualquer outro conteúdo gerado por IA.
  - Dado que eu recuso o consentimento, paro de receber respostas geradas por IA.

**US-9**: Como paciente, eu quero conseguir chegar a uma pessoa real quando minha situação não é algo que um bot deveria tratar, para que eu me sinta seguro para perguntar qualquer coisa.
- Critérios de aceite:
  - Dado que eu peço explicitamente um humano, quando o pedido é processado, então sou avisado de que uma pessoa vai assumir, e um humano recebe minha conversa com contexto.

## Administrador ZapCare (operador)

**US-10**: Como administrador do ZapCare, eu quero mudar o status de cobrança de uma clínica a partir de um único lugar confiável, para que o estado de cobrança nunca possa ser alterado por uma ação não autorizada.
- Critérios de aceite:
  - Dado que estou autenticado como operador, quando eu atualizo o status de cobrança de uma clínica, então a mudança é aplicada e registrada em log.
  - Dado que qualquer outro chamador tenta a mesma operação no banco, ela é rejeitada.

**US-11**: Como administrador do ZapCare, eu quero uma visão de CRM do pipeline comercial, para que eu consiga acompanhar leads desde o primeiro contato até virar cliente pagante sem sair da ferramenta que uso para operar.
- Critérios de aceite:
  - Dado que um lead é criado (landing page, fluxo de diagnóstico ou entrada manual), quando eu abro a visão de CRM, então o lead aparece com sua origem e estágio atual.
  - Dado que eu atualizo o estágio de um lead, a mudança se reflete imediatamente na visão de pipeline.

**US-12**: Como administrador do ZapCare, eu quero reenfileirar manualmente uma mensagem falha, para que uma falha passageira não deixe um paciente sem resposta indefinidamente.
- Critérios de aceite:
  - Dado que uma mensagem está em estado de falha, quando eu disparo um reenfileiramento pelo painel admin, então ela reentra no processamento e a ação é registrada em log.

## Operador de suporte

**US-13**: Como operador de suporte, eu quero ser alertado automaticamente quando uma integração cai, para que eu consiga responder antes que uma clínica perceba.
- Critérios de aceite:
  - Dado que um health check agendado detecta uma dependência falhando, quando a falha não está dentro da janela de cooldown, então eu recebo um alerta identificando qual dependência falhou.

**US-14**: Como operador de suporte, eu quero ver quando uma transferência humana fica sem resposta por tempo demais, para que nenhuma conversa de paciente pare silenciosamente.
- Critérios de aceite:
  - Dado que uma conversa transferida ultrapassa o limite de SLA, quando o SLA monitor roda, então eu sou alertado e a conversa é sinalizada.

## Desenvolvedor / mantenedor do sistema

**US-15**: Como desenvolvedor que mantém este sistema, eu quero as dependências de cada workflow documentadas, para que eu consiga depurar ou repassar um workflow sem ter que fazer engenharia reversa do zero.
- Critérios de aceite:
  - Dado que um workflow toca uma tabela de banco, uma RPC ou uma credencial externa, quando eu leio a documentação dele, então essa dependência está listada.

**US-16**: Como desenvolvedor que mantém este sistema, eu quero staging isolado de produção por mais do que convenção, para que uma mudança experimental nunca chegue a um paciente real.
- Critérios de aceite:
  - Dado que um workflow roda no ambiente de staging, quando ele processa uma mensagem, então só prossegue para números de telefone em uma allowlist explícita.

**US-17**: Como desenvolvedor que mantém este sistema, eu quero mudanças de schema de banco de dados rastreadas como migrações, para que o histórico do schema seja reconstruível e revisável.
- Critérios de aceite:
  - Dado que uma coluna ou tabela nova é necessária, quando ela é adicionada ao banco em produção, então existe um arquivo de migração correspondente versionado.
  - *(Lacuna conhecida: ainda não é verdade para `crm_leads`/`crm_notes` — ver [Arquitetura § 9](arquitetura.md#9-débito-arquitetural-conhecido).)*
