# ADR-0006: Separar Painel Admin, Landing Page e Workflows em Deployáveis Independentes

> 🇺🇸 [Read in English](../../en/adrs/0006-admin-landing-workflows-separation.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

O sistema tem três preocupações bem diferentes: ferramenta interna de operação (painel admin), marketing/captura de lead pública (landing page), e a lógica de automação/orquestração (workflows n8n). Essas têm públicos diferentes (operador vs. visitantes públicos vs. o próprio sistema em execução), perfis de risco diferentes (um bug na landing page é constrangedor; um bug de workflow pode mandar a mensagem errada para um paciente real), e cadências de deploy diferentes.

## Decisão

Manter o painel admin, a landing page e os workflows do n8n como três unidades independentemente deployáveis, coordenadas apenas através do banco de dados Supabase compartilhado — nunca compartilhando codebase, pipeline de deploy, ou chamadas diretas de serviço para serviço.

## Alternativas consideradas

- **Um único app web monolítico servindo tanto as páginas públicas de marketing quanto a área admin autenticada**: menos peças móveis para deployar, mas acopla o risco de uptime/deploy do site público ao da ferramenta admin, e mistura uma superfície pública não autenticada com uma interna autenticada no mesmo codebase — uma fronteira de segurança pior.
- **Workflows do n8n chamando a API do painel admin diretamente (ou vice-versa)**: acoplamento mais forte permitiria interação em tempo real mais rica, mas quebra o princípio de "o banco de dados é o contrato" (ver [Arquitetura § 1](../arquitetura.md#1-visão-geral)), fazendo cada lado depender do uptime e da estabilidade de API do outro, em vez de depender só do Supabase.
- **Três deployáveis independentes coordenados via o banco de dados (escolhido)**: a landing page pode ser redeployada sem tocar nos workflows; um workflow pode ser editado sem redeployar nenhum dos dois apps web; e uma indisponibilidade em um deles (ex: o painel admin cai) não derruba o atendimento de conversas voltado ao paciente.

## Consequências

**Positivas**:
- Contenção clara de raio de impacto: um bug na landing page não pode quebrar conversas de paciente; um bug de workflow não pode corromper o site de marketing.
- Cada peça pode ser construída, testada e deployada em seu próprio ritmo — relevante para um contexto de desenvolvedor solo alternando entre trabalho de "material comercial" e trabalho de "produto principal".
- O padrão de banco-de-dados-como-contrato mantém as três peças honestas sobre como os dados realmente são, já que não existe uma API interna privada para mascarar um descompasso de schema.

**Negativas**:
- Nenhum código compartilhado entre o painel admin e a landing page apesar de ambos serem apps React/Vite/Tailwind com necessidades utilitárias sobrepostas (ex: formatação de número de telefone) — existe alguma duplicação em vez de um pacote interno compartilhado. `[a validar: extensão da duplicação]`
- Coordenar uma mudança que atravessa, digamos, um novo campo de CRM tocado tanto pelo formulário de lead da landing page quanto pela visão de CRM do painel admin, exige atualizar dois codebases mais o schema compartilhado, em vez de um só.

## Relacionados

[Arquitetura](../arquitetura.md), [Requisitos de Sistema § 6 (Manutenção)](../requisitos-de-sistema.md#6-requisitos-de-manutenção)
