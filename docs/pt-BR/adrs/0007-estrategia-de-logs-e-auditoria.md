# ADR-0007: Logs Estruturados e Trilhas de Auditoria Explícitas em vez de Escritas Silenciosas

> 🇺🇸 [Read in English](../../en/adrs/0007-logging-and-audit-strategy.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

Um sistema operado por uma única pessoa, sem equipe dedicada de operações ou QA, vai falhar de formas que ninguém está observando em tempo real, a menos que falhas e ações sensíveis sejam tornadas visíveis por design. Existem duas necessidades diferentes aqui: (1) log operacional, para que falhas possam ser diagnosticadas depois do fato, e (2) trilhas de auditoria para ações sensíveis (mudanças de cobrança, pausa/retomada de IA, consentimento), que precisam responder "quem fez o quê, quando" mesmo que o operador seja o único "quem" hoje.

## Decisão

- Registrar todo evento de erro/aviso/informação de workflow numa tabela `logs` estruturada, etiquetada por nome de workflow, nome de nó e identificadores relevantes (telefone, ID de mensagem) — não apenas no histórico de execução do próprio n8n, que é mais difícil de consultar e tem seus próprios limites de retenção.
- Tratar registros de consentimento como registros de auditoria permanentes e imutáveis, nunca sujeitos à limpeza padrão de retenção de log.
- Rotear toda ação sensível do painel admin (pausar IA, reenfileirar uma mensagem, mudar status de cobrança) por caminhos de código que gravam uma entrada de log explícita, em vez de um update de banco puro sem rastro.

## Alternativas consideradas

- **Depender só do log de execução nativo do n8n**: zero trabalho extra, mas logs de execução são escopados por workflow e não são facilmente consultáveis/combináveis com dados de negócio (ex: "mostre todo erro para essa clínica em todos os workflows"), e ficam sujeitos às configurações de retenção do próprio n8n, não a uma política de retenção deliberada e orientada a negócio.
- **Registrar tudo indefinidamente**: política de retenção mais simples, mas conflita diretamente com o princípio de minimização de dados da LGPD aplicado em outras partes do sistema (ver [Regras de Negócio § 8](../regras-de-negocio.md#8-regras-de-segurança-e-privacidade)) — a maioria dos logs operacionais não precisa viver para sempre, e alguns (como conteúdo de mensagem em logs de erro) são exatamente o tipo de dado que deveria ser minimizado.
- **Tabela de logs estruturada com retenção diferenciada, mais escritas explícitas de trilha de auditoria para ações sensíveis (escolhido)**: dá diagnosticabilidade consultável e cruzada entre workflows, mantém registros genuinamente permanentes (consentimento) permanentes, e deixa todo o resto expirar num cronograma definido.

## Consequências

**Positivas**:
- Falhas são diagnosticáveis por workflow e nó sem precisar abrir a interface do n8n e caçar no histórico de execução.
- Ações administrativas sensíveis têm um rastro — relevante tanto para a própria responsabilização do operador quanto como base caso um segundo operador/membro de equipe seja adicionado algum dia.
- A retenção de log é uma política deliberada e documentada, não "o que a ferramenta define por padrão".

**Negativas**:
- Isso é uma disciplina manual, imposta pela forma como cada ação de workflow/painel é escrita, não algo que um framework garante automaticamente — um workflow novo que esquece de registrar um erro é um risco real e recorrente, não hipotético.
- A própria limpeza de retenção é um job agendado (parte do Retry Scheduler) que, se falhasse silenciosamente, deixaria os logs crescerem sem limite sem que ninguém percebesse de imediato — `[a validar: se a falha da própria limpeza é monitorada]`.

## Relacionados

[Arquitetura § 6, § 7](../arquitetura.md), [Regras de Negócio § 8](../regras-de-negocio.md#8-regras-de-segurança-e-privacidade), [Requisitos de Sistema § 4 (Observabilidade)](../requisitos-de-sistema.md#4-requisitos-de-observabilidade)
