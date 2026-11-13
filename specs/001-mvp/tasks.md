# Arc Charge Agent — Tarefas (tasks.md)

Feature: `001-mvp` · Depende de: `spec.md` e `plan.md`

Formato: `[ID] [P?] [História] Descrição`. `[P]` indica tarefa que pode correr em paralelo.

## Pré-requisitos

- [ ] T000 Acesso ao Arc Studio com a conta Circle do autor (login feito por ele)
- [ ] T001 Decidir tamanho da etapa de energia e prazo padrão da sessão

## Fase A — Contratos no Arc Studio (US1, US4)

- [ ] T010 [US1] Pedir o plano de `ChargerRegistry` em modo Plan e revisar
- [ ] T011 [US1] Gerar `ChargerRegistry` com testes
- [ ] T012 [US4] Pedir o plano de `ChargeSessions` em modo Plan e revisar
- [ ] T013 [US4] Gerar `ChargeSessions` com testes, incluindo assinatura EIP-712
- [ ] T014 Revisar os achados das análises do Arc Studio
- [ ] T015 Publicar os dois contratos na testnet da Arc, com aprovação do autor
- [ ] T016 Registrar endereços e transações de deploy

## Fase B — Contratos no repositório (US1, US3, US4)

- [ ] T020 Trazer o código gerado para o repositório, sem comentários
- [ ] T021 [P] Testes unitários do cadastro
- [ ] T022 [P] Testes de assinatura: válida, assinante errado, sessão errada, acima do orçamento, incoerente com a energia
- [ ] T023 [P] Testes de prazo: liquidar depois e recuperar antes
- [ ] T024 Fuzz da divisão e invariante da soma dos créditos (SC-002)
- [ ] T025 Rodar com `arc-forge test`
- [ ] T026 Rodar o Arc Porter e tratar os achados (SC-004)
- [ ] T027 Revisão de segurança do squad

## Fase C — Simulador e agente (US2, US3)

- [ ] T030 [US3] Simulador: entrega em etapas, exige comprovante novo a cada etapa, guarda o mais recente
- [ ] T031 [US3] Simulador liquida a sessão ao final
- [ ] T032 [US2] Agente: regras de preço máximo, orçamento, energia desejada e carregadores permitidos
- [ ] T033 [US2] Agente escolhe o carregador mais barato dentro das regras ou explica por que não abriu sessão
- [ ] T034 [US3] Agente confere a energia informada e assina o comprovante acumulado
- [ ] T035 Sessão completa na testnet com 10 etapas (SC-001)
- [ ] T036 [P] Cenário: agente para de assinar
- [ ] T037 [P] Cenário: carregador para de entregar
- [ ] T038 [P] Cenário: operador não liquida e o pagador recupera o orçamento

## Fase D — Painel (US5)

- [ ] T040 [US5] Lista de carregadores com preço e divisão
- [ ] T041 [US5] Sessão: energia, valor, divisão e links do explorador
- [ ] T042 [US5] Avisos visíveis de "simulado" e "testnet"

## Fase E — Depois do MVP (US6)

- [ ] T050 Registrar agente e carregador com ERC-8004
- [ ] T051 Trocar os comprovantes por Circle Nanopayments com x402

## Dependências

- T000 bloqueia a fase A.
- A fase A bloqueia B; B bloqueia C; D depende dos endereços da fase A.
