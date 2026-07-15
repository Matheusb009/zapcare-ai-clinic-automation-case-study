# ADR-0002: Usar Supabase como Camada de Backend/Banco de Dados

> 🇺🇸 [Read in English](../../en/adrs/0002-supabase-as-backend.md)

- **Status**: Aceito
- **Data**: `[a validar: data exata da decisão]` — fase inicial de construção do projeto

## Contexto

O sistema precisa de um banco de dados relacional (dados estruturados e relacionais: clínicas, conversas, agendamentos, fila de mensagens) que possa ser escrito pelos workflows do n8n (backend confiável), e também precisa ser lido/escrito diretamente por duas aplicações frontend (painel admin, landing page) sem necessariamente construir e hospedar uma API REST/GraphQL separada para cada uma.

## Decisão

Usar Supabase (PostgreSQL gerenciado com Row-Level Security, acesso REST/RPC autogerado, e subscriptions Realtime) como o backend único tanto para a camada de automação quanto para as duas aplicações web.

## Alternativas consideradas

- **PostgreSQL autogerenciado + uma camada de API customizada (ex: Express/FastAPI)**: controle total sobre a superfície de API, mas exige construir, hospedar e proteger um serviço de backend separado, além de sua própria camada de autenticação — escopo extra relevante para uma construção solo.
- **Firebase/Firestore**: conveniência comparável de "backend-as-a-service", mas um modelo de documentos é um encaixe pior para dados genuinamente relacionais como os deste projeto (clínicas → conversas → agendamentos, com chaves estrangeiras reais e constraints como unicidade de `(telefone, clinic_id)`).
- **Supabase (escolhido)**: Postgres relacional (melhor encaixe para o formato dos dados), Row-Level Security aplicada na camada do banco (então as mesmas regras de segurança valem independente de qual cliente conecta), APIs autogeradas eliminando a necessidade de uma camada REST construída à mão, e Realtime para UI que atualiza ao vivo onde necessário.

## Consequências

**Positivas**:
- Nenhuma API de backend separada foi necessária para nenhum dos dois frontends — ambos falam com o Supabase diretamente, cortando tempo de construção real.
- RLS empurra o modelo de segurança para dentro do próprio banco de dados, então um bug no código frontend não pode expor dados que uma política não permite, independente de em qual app o bug está.
- RPCs `SECURITY DEFINER` dão um padrão limpo para escalonamento de privilégio estreito e auditável (ex: a RPC de status de cobrança) sem dar acesso amplo de escrita a nenhum papel de cliente.

**Negativas**:
- A corretude das políticas de RLS vira todo o modelo de segurança para escritas públicas (ex: a chave anônima da landing page) — uma política mal configurada é um risco direto de exposição de dados, concentrado em uma única camada em vez de distribuído por validação em nível de API.
- A disciplina de evolução de schema (migração para toda mudança) é um processo que a própria equipe precisa impor; já falhou uma vez (`crm_leads`/`crm_notes` criadas diretamente no banco em produção — ver [Arquitetura § 9](../arquitetura.md#9-débito-arquitetural-conhecido)).
- Dependência de fornecedor especificamente no Supabase para as convenções de Realtime/RLS/RPC — migrar dele significaria reimplementar lógica de backend relevante, não apenas trocar uma connection string.

## Relacionados

[Arquitetura § 3.3](../arquitetura.md#33-camada-de-dados-supabase--postgresql), [Integrações § 4](../integracoes.md#4-supabase-banco-de-dados--backend)
