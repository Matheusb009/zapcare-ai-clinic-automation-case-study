# ADR-0003: Usar Chatwoot para Transferência Humana

> 🇺🇸 [Read in English](../../en/adrs/0003-chatwoot-for-handoff.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

Quando a Sofia transfere uma conversa para um humano, a equipe da clínica precisa de um lugar para ver a conversa, responder ao paciente e marcá-la como resolvida — algo utilizável por uma recepção não-técnica, não uma visão crua de banco de dados ou uma ferramenta de desenvolvedor.

## Decisão

Usar Chatwoot (self-hosted, open-source) como a caixa de suporte voltada para humanos, para todas as conversas escaladas.

## Alternativas consideradas

- **Construir uma "caixa de transferência" customizada dentro do painel admin**: controle total sobre a UX, mas duplica uma quantidade grande de funcionalidade (caixa de entrada multi-conversa, atribuição, labels, estado lido/não lido, rastreamento de resolução) que já existe em ferramentas de suporte maduras — um mau uso do tempo de construção solo para uma funcionalidade que não diferencia o produto.
- **Um SaaS comercial de helpdesk (ex: Zendesk, Intercom)**: maduro e polido, mas o preço de SaaS por assento/conversa é um mau encaixe nesta fase, e a maior parte da profundidade de funcionalidades deles (workflows de ticket, SLAs em muitos canais) é desnecessária aqui.
- **Chatwoot (escolhido)**: open-source e self-hostable (sem custo por assento nessa escala), tem uma API de webhook para eventos de resolução (necessária para reativar a IA automaticamente), e sua interface já é familiar para muita equipe de suporte por se parecer com ferramentas de helpdesk comuns.

## Consequências

**Positivas**:
- Evitou construir uma UI de caixa de entrada do zero — redução real de escopo para uma construção solo.
- O loop orientado a webhook de "resolvido → reativa IA" (ver [Arquitetura § 5](../arquitetura.md#5-fluxo-de-dados-transferência-para-humano)) foi direto de implementar porque o Chatwoot expõe eventos de estado de conversa, não só uma interface.
- Sem custo recorrente de SaaS por assento conforme mais clínicas são adicionadas.

**Negativas**:
- Self-hosting significa que o ZapCare — não um fornecedor — é dono do uptime, patching e backups do Chatwoot.
- Uma indisponibilidade do Chatwoot remove todo o caminho de transferência humana; é por isso que ele está incluído no health check automatizado (ver [Arquitetura § 6](../arquitetura.md#6-mecanismos-de-confiabilidade)).

## Relacionados

[Arquitetura § 3.4](../arquitetura.md#34-transferência-humana-chatwoot), [Regras de Negócio § 3](../regras-de-negocio.md#3-regras-de-transferência-humana-handoff), [Integrações § 2](../integracoes.md#2-chatwoot-transferência-humana--caixa-de-suporte)
