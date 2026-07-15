# ADR-0008: Cobrança Manual Antes de Cobrança Automatizada (Stripe)

> 🇺🇸 [Read in English](../../en/adrs/0008-manual-billing-before-stripe.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — documentado junto com a implementação de cobrança

## Contexto

O produto precisa de uma forma de cobrar clínicas e rastrear o estado trial/ativo/atrasado/cancelado. Com poucas clínicas, integrar uma plataforma de pagamento (Stripe ou um equivalente local) significa engenharia e overhead de compliance reais — tratamento de webhook, reconciliação, emissão de nota, tratamento de falha/disputa — antes de haver volume de cobrança suficiente para justificar isso.

## Decisão

Rodar a cobrança manualmente: o operador acompanha o pagamento (via PIX/boleto, fora do produto) e atualiza o status de cobrança de cada clínica através de uma única ação administrativa restrita. Adiar explicitamente Stripe, emissão automatizada de nota, cobrança recorrente automática e um portal de cobrança voltado ao cliente até que o número de clínicas justifique o custo de integração.

## Alternativas consideradas

- **Integrar Stripe (ou um provedor de cobrança local) desde o início**: automatizado, escala de forma limpa, mas representa esforço real de engenharia antecipado (tratamento de webhook, reconciliação de estado de pagamento, lógica de falha/retry, provavelmente uma revisão de compliance) para uma fase em que talvez existam poucas clínicas — custo alto em relação ao número de transações que de fato processaria neste momento.
- **Nenhum rastreamento de status de cobrança (informal, fora do sistema)**: custo zero de engenharia, mas deixa o painel admin incapaz de refletir o estado real de pagamento de uma clínica, e remove a capacidade de bloquear/avisar sobre contas atrasadas dentro da experiência do produto.
- **Status de cobrança manual, rastreado no produto, aplicado por uma única RPC restrita (escolhido)**: entrega os benefícios voltados ao produto (banners de trial, avisos de atraso, um `billing_status` consultável por clínica) sem o custo de uma integração de pagamento, mantendo o caminho de escrita estreito e auditável para que "manual" não signifique "inseguro".

## Consequências

**Positivas**:
- Evitou custo real de integração e superfície de compliance antes de haver volume de cobrança que justificasse isso — uma decisão deliberada de sequenciamento, não um atalho tomado sem cuidado (está documentada, com um gatilho definido para revisitar: clínicas pagantes suficientes).
- O padrão de RPC de status de cobrança (um único caminho de escrita restrito) significa que a cobrança "manual" ainda tem a mesma disciplina de segurança de qualquer outra coisa no sistema — ver [Arquitetura § 7](../arquitetura.md#7-fronteiras-de-segurança).
- As clínicas ainda têm uma experiência de produto real em torno de cobrança (contagem regressiva de trial, aviso de atraso) mesmo que a cobrança do pagamento em si aconteça fora do sistema.

**Negativas**:
- Toda mudança de status de cobrança exige atenção manual do operador — isso não escala além de um número relativamente pequeno de clínicas sem virar, ele mesmo, um gargalo operacional.
- Nenhuma régua de cobrança automatizada, nenhum histórico de fatura self-service para a clínica, nenhuma lógica de cobrança proporcional — tudo explicitamente fora de escopo por ora (ver [Regras de Negócio § 5](../regras-de-negocio.md#5-regras-de-cobrança--trial)).
- A eventual migração para cobrança automatizada vai exigir trabalho de design real (mapear os estados manuais atuais para o ciclo de vida de assinatura do Stripe) — custo adiado, não eliminado.

## Relacionados

[Regras de Negócio § 5](../regras-de-negocio.md#5-regras-de-cobrança--trial), [Roadmap](../roadmap.md), [Caso de Uso UC-7](../casos-de-uso.md#uc-7--admin-altera-o-status-de-cobrança-de-uma-clínica)
