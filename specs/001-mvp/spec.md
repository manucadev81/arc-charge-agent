# Arc Charge Agent — Especificação do MVP (spec.md)

Feature: `001-mvp` · Estado: no backlog, aguardando início · Data: 13/11/2026
Autor: Manoel Lúcio (Nearxlabs)

Um agente paga a recarga de um carro elétrico por kWh entregue, em USDC, na Arc. Este MVP prova o fluxo de ponta a ponta com um carregador simulado, na testnet.

## 1. Princípios

1. **Simulado e declarado.** O carregador é um simulador. Nenhum texto do projeto sugere hardware ou cliente real.
2. **Só testnet.** Nada vai para mainnet sem revisão independente e decisão do autor.
3. **Fatos da Arc vêm dos docs.** Todo comportamento da rede citado aponta para a página oficial.
4. **Contratos pelo Arc Studio.** Os contratos são escritos, testados e publicados na testnet pelo Arc Studio.
5. **Código sem comentários.** Nenhum comentário em código, em nenhuma linguagem.

## 2. Problema e aposta

- Quem carrega em várias redes lida com um app, um cartão e uma fatura por rede. Para frotas isso vira trabalho de conciliação.
- Operadores assumem risco de pagamento ou exigem pré-autorização.
- A receita de cada sessão é dividida à mão, no fim do mês, entre operador e dono do local.

Aposta: um agente com orçamento e regras paga conforme a energia chega, e cada sessão já sai dividida entre as partes.

Fora da aposta deste MVP: provar adoção por operadores reais.

## 3. Personas

| Persona | Papel no MVP |
| :- | :- |
| Gestor de frota ou motorista | Define orçamento e regras do agente |
| Agente pagador | Escolhe o ponto, abre a sessão, paga por etapa, encerra |
| Operador do ponto | Cadastra o carregador e o preço, recebe sua parte |
| Dono do local | Recebe sua parte de cada sessão |

## 4. Histórias de usuário

### US1 (P1) — Operador cadastra um carregador
- **Dado** um operador, **quando** cadastra um carregador com preço por kWh, endereço do dono do local e percentual dele, **então** o carregador aparece como disponível com esses dados.
- **Dado** um percentual do dono do local acima de 100%, **então** o cadastro é recusado.

### US2 (P1) — Agente carrega dentro das regras
- **Dado** regras (preço máximo por kWh, orçamento da sessão, energia desejada), **quando** o agente inicia, **então** escolhe o carregador disponível mais barato que respeita o preço máximo.
- **Dado** nenhum carregador dentro do preço máximo, **então** o agente não abre sessão e explica o motivo.
- **Dado** uma sessão aberta, **então** o orçamento fica travado em USDC e nunca é ultrapassado.

### US3 (P1) — Pagamento acompanha a energia
- **Dado** uma sessão em andamento, **quando** o carregador entrega uma etapa de energia, **então** o agente assina um comprovante com o total devido até ali, e só então a próxima etapa é entregue.
- **Dado** que o agente para de assinar, **então** o carregador para de entregar em no máximo uma etapa.
- **Dado** que o carregador para de entregar, **então** o agente não assina mais nada e o prejuízo máximo é zero etapas pagas sem entrega.

### US4 (P1) — Liquidação com divisão automática
- **Dado** o último comprovante, **quando** o operador liquida a sessão, **então** operador e dono do local recebem crédito conforme os percentuais, e o orçamento não usado volta como crédito ao pagador.
- **Dado** uma sessão que o operador nunca liquidou, **quando** o prazo expira, **então** o pagador recupera o orçamento inteiro.
- **Dado** um comprovante com valor acima do orçamento ou incompatível com a energia e o preço, **então** a liquidação é recusada.

### US5 (P2) — Painel da sessão
- **Dado** uma sessão, **então** vejo energia entregue, valor pago, divisão entre as partes e links das transações no explorador.

### US6 (P3) — Identidade e pagamentos em escala
- Registrar agente e carregador com ERC-8004.
- Trocar os comprovantes próprios por Circle Nanopayments com x402.

## 5. Requisitos funcionais

- **FR-001** Cadastro de carregadores com operador, dono do local, percentual do dono do local em pontos-base, preço por kWh em USDC e estado ativo ou inativo.
- **FR-002** Abertura de sessão com orçamento, preço máximo aceito e prazo; o preço fica travado no valor do cadastro naquele momento.
- **FR-003** Comprovante assinado pelo agente no padrão EIP-712 com sessão, energia acumulada e valor acumulado.
- **FR-004** A chave que assina os comprovantes pode ser diferente da carteira que financia a sessão.
- **FR-005** Liquidação pelo operador com o último comprovante; verificação da assinatura, do orçamento e da coerência entre energia, preço e valor.
- **FR-006** Créditos por saque individual; um destinatário que não pode receber USDC não bloqueia os demais.
- **FR-007** Recuperação do orçamento pelo pagador após o prazo, se não houve liquidação.
- **FR-008** Simulador de carregador que entrega energia em etapas e exige comprovante novo a cada etapa.
- **FR-009** Agente com regras de preço máximo, orçamento por sessão, energia desejada e lista de carregadores permitidos.
- **FR-010** Painel com sessões, energia, valores e links do explorador.

## 6. Requisitos não funcionais

- **NFR-001** Só a interface ERC-20 do USDC (6 casas decimais); nenhum uso de `msg.value`.
- **NFR-002** Sem aleatoriedade vinda de `block.prevrandao`.
- **NFR-003** Limites em todo laço.
- **NFR-004** Uma transação para abrir e uma para liquidar por sessão; as etapas intermediárias não vão para a rede.
- **NFR-005** Chaves e credenciais fora do repositório.

## 7. Fora do escopo

Hardware real, protocolos de carregador em produção, conversão de USDC para reais, cadastro de clientes, mainnet, venda de participação em receita para investidores.

## 8. Critérios de sucesso

- **SC-001** Uma sessão completa na testnet: abrir, 10 etapas de energia, liquidar, sacar, com links das transações.
- **SC-002** A soma creditada a operador, dono do local e pagador é igual ao orçamento travado, em todos os testes.
- **SC-003** Demonstração de cada falha: agente para de assinar, carregador para de entregar, operador não liquida.
- **SC-004** Contratos passam sem achado de severidade alta no Arc Porter.

## 9. Decisões em aberto

- **[DECIDIR-1]** Tamanho da etapa de energia (por exemplo, 100 Wh).
- **[DECIDIR-2]** Prazo padrão de uma sessão.
- **[DECIDIR-3]** Fase 2 com Circle Nanopayments exige conta e chaves no Circle Console: quando criar.
- **[DECIDIR-4]** Buscar um operador de recarga parceiro para validar depois do MVP.

## 10. Fontes (conferidas em 13/11/2026)

- https://docs.arc.io/build/agentic-economy.md
- https://docs.arc.io/build/sample-apps/arc-nanopayments.md
- https://docs.arc.io/arc/references/evm-differences.md
- https://docs.arc.io/arc/references/contract-addresses.md
- https://docs.arc.io/ai/arc-studio.md
- https://www.arc.io/blog/how-openvpp-is-exploring-realtime-energy-settlement-on-arc
