# ADR-0004: UazAPI vs. a API Oficial do WhatsApp Business

> 🇺🇸 [Read in English](../../en/adrs/0004-uazapi-vs-official-whatsapp-api.md)

- **Status**: Aceito, com revisão futura planejada
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

Toda conversa que o produto existe para automatizar acontece no WhatsApp. Existem dois caminhos amplos para enviar/receber mensagens de WhatsApp programaticamente: a API oficial do WhatsApp Business (via um Business Solution Provider aprovado pela Meta) ou um gateway terceiro que conecta a um número de WhatsApp existente fora do caminho de integração oficialmente sancionado pela Meta (ex: UazAPI).

## Decisão

Usar a UazAPI para o piloto e a fase inicial de clínicas. Tratar a API oficial como uma migração futura planejada, não uma opção permanentemente rejeitada.

## Alternativas consideradas

- **API oficial do WhatsApp Business via um BSP**: totalmente em conformidade, feita para escala, mas exige verificação de negócio na Meta, um número registrado/dedicado, templates de mensagem pré-aprovados para contato iniciado pela empresa e, tipicamente, um contrato com um BSP — dias a semanas de setup e um piso de custo mais alto, tudo isso antes de validar se uma única clínica sequer usaria o produto.
- **UazAPI (escolhida por ora)**: conecta um número de WhatsApp já existente da clínica através de um gateway hospedado em horas, com um piso de custo muito mais baixo — mas opera fora do caminho de integração oficialmente sancionado pelo WhatsApp, carregando risco real de restrição do número se os padrões de uso forem sinalizados como automatizados ou abusivos.

## Consequências

**Positivas**:
- A primeira clínica-piloto pôde entrar no ar em horas em vez de semanas, o que importou quando o objetivo era validar valor do produto, não construir para uma escala que ainda não existia.
- Custo fixo menor cabe numa fase pré-receita/receita inicial.

**Negativas / risco aceito**:
- Risco real, reconhecido, de o WhatsApp restringir ou banir o número conectado se padrões de uso forem sinalizados — isso é um risco de continuidade de negócio, não apenas um incômodo técnico, e é explicitamente rastreado, não ignorado.
- Nenhum SLA ou caminho de suporte respaldado pela Meta se algo der errado no nível da plataforma.

**Mitigação / caminho de migração**:
- Todo I/O de WhatsApp está isolado nos workflows Receptor/Worker/Lembretes em vez de espalhado pelo código — uma escolha de design deliberada especificamente para que migrar para a API oficial depois seja uma troca de provedor na fronteira de integração, não uma reescrita de arquitetura.
- A decisão é explicitamente delimitada a "agora, nesta escala" — ver [Integrações § 1](../integracoes.md#1-mensageria-whatsapp-api-oficial-do-whatsapp-business-vs-uazapi) para a comparação completa e o gatilho planejado para revisitá-la (número de clínicas e receita justificando custo e setup de BSP).

## Relacionados

[Integrações § 1](../integracoes.md#1-mensageria-whatsapp-api-oficial-do-whatsapp-business-vs-uazapi), [Arquitetura](../arquitetura.md)
