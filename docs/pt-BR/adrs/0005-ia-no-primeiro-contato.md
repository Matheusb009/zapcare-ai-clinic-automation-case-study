# ADR-0005: Usar IA no Primeiro Contato, com Modelos em Camadas e uma Fronteira Rígida de Transferência

> 🇺🇸 [Read in English](../../en/adrs/0005-ai-first-contact.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

Clínicas precisam que toda mensagem de paciente recebida seja respondida prontamente, mas uma recepcionista não consegue estar online 24/7, e contratar mais equipe para cobrir volume é exatamente o problema de custo que o produto existe para resolver. A alternativa a um modelo humano-primeiro é um modelo IA-primeiro — mas conversas adjacentes à saúde carregam risco real se a IA se exceder (alegações clínicas, compromissos de preço, promessas de procedimento).

## Decisão

Deixar uma assistente de IA (Sofia) atender o primeiro contato e a conversa de rotina por padrão, apoiada por modelos em camadas (uma camada rápida/barata para perguntas simples, uma camada mais robusta para as complexas), com limites rígidos e aplicados em código sobre o que ela pode fazer, e transferência incondicional e imediata para um humano em qualquer coisa fora desses limites.

## Alternativas consideradas

- **Humano-primeiro, IA-assistida (IA rascunha respostas, um humano aprova/envia)**: mais seguro em princípio, mas reintroduz exatamente o gargalo de equipe que o produto deveria remover — um humano ainda precisa estar online para aprovar cada mensagem.
- **IA-primeiro com uma única camada de modelo**: mais simples de construir e raciocinar, mas ou paga demais em perguntas simples (sempre usando o modelo mais forte) ou performa mal nas complexas (sempre usando o modelo barato) — nenhuma camada única cobria bem as duas pontas da faixa de complexidade de forma econômica.
- **IA-primeiro, em camadas, com limites comportamentais rígidos e transferência automática (escolhido)**: mantém a responsividade sempre ativa que resolve o problema central, controla custo via camadas, e trata "a IA não deve decidir isso" como uma decisão de roteamento aplicada pela lógica do Worker/transferência — não apenas uma instrução de prompt que o modelo poderia ignorar sob pressão.

## Consequências

**Positivas**:
- Pacientes recebem uma primeira resposta imediata e sempre disponível — atacando diretamente o problema central (ver [README](../../../README.pt-BR.md#o-problema)).
- O custo por conversa fica controlado porque a maioria das perguntas roteia para a camada de modelo mais barata; só as genuinamente complexas escalam.
- A fronteira de transferência é aplicada na lógica de workflow (ver [Regras de Negócio § 2, § 3](../regras-de-negocio.md)), não só no prompt, então um modelo se comportando de forma inesperada ainda assim não pode, por exemplo, continuar uma conversa depois que um humano assumiu — a pausa é uma checagem de estado no banco de dados, não uma sugestão ao modelo.

**Negativas**:
- Duas camadas de modelo significam dois conjuntos de comportamento para validar e monitorar quanto a desvio ao longo do tempo, em vez de um.
- Hoje não existe uma suíte de avaliação (eval) automatizada para qualidade conversacional — mudanças de prompt/modelo são validadas manualmente contra staging, o que não escala indefinidamente conforme a complexidade dos fluxos cresce (sinalizado em [Integrações § 5](../integracoes.md#5-a-camada-de-ia-llm--speech-to-text)).
- Uma indisponibilidade completa do provedor de LLM trava todas as conversas de uma vez, sem provedor de fallback hoje.

## Relacionados

[Fluxos da Sofia IA](../fluxos-sofia-ia.md), [Regras de Negócio § 2, § 3](../regras-de-negocio.md), [Integrações § 5](../integracoes.md#5-a-camada-de-ia-llm--speech-to-text)
