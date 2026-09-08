# wl-reviewer

> Revisão arquitetural de artefatos O2P por 4 agentes especialistas em paralelo.

Uma skill para [Claude Code](https://claude.ai/code) que replica o processo de revisão de fluxos de negócio White Label por múltiplos arquitetos — de forma automatizada, estruturada e repetível.

---

## O que faz

Dado um artefato `.md` de fluxo O2P (Order-to-Payment), a skill despacha **3 agentes especialistas em paralelo** e um **árbitro CTO** que consolida e veredicta:

| Agente | Perspectiva |
|--------|-------------|
| 🔧 Arq. Técnico Salesforce | Implementabilidade em Comms Cloud — DRO, SOM, SF Objects, testabilidade |
| 📡 Arq. Funcional TM Forum | Conformidade com TMF Open APIs, eTOM v2, ODA, estados canônicos |
| 🏛️ Arq. Estratégico TOGAF | Governança de ADRs, rastreabilidade, vocabulário, concerns de compliance |
| ⚖️ Validação CTO | Árbitro cross — confirma, modera ou refuta cada achado com base nos documentos de referência |

**Output:** tabela de achados com trecho original, observação por agente, veredito CTO, severidade (🔴/🟡/🟢) e ação sugerida — com coluna vazia para o 5º arquiteto humano preencher.

**Veredito final:** `GO` · `GO-CONDICIONAL` · `NO-GO`

---

## Pré-requisitos

- [Claude Code](https://claude.ai/code) com acesso ao projeto ADP
- Repo ADP clonado com as ADRs de referência (`architecture/decisions/`)
- Artefato `.md` no formato `flows-wiki` com frontmatter, diagrama Mermaid, tabela de Steps e Notes

---

## Instalação

Copie o arquivo da skill para a pasta `.claude/skills/` do seu projeto ADP:

```bash
cp review-o2p-flow.md <seu-projeto>/.claude/skills/
```

A skill será descoberta automaticamente pelo Claude Code na próxima sessão.

---

## Como usar

No Claude Code, dentro do projeto ADP:

```
/review-o2p-flow architecture/flows-wiki/o2p/O2P-Fulfillment.md
```

Ou para um artefato fora do diretório padrão:

```
/review-o2p-flow /caminho/absoluto/para/MeuFluxo.md
```

---

## O que a skill lê antes de revisar

A skill carrega automaticamente os seguintes documentos de referência do seu projeto antes de despachar os agentes:

- `SOUL.md` — non-negotiables arquiteturais (#7: nunca sobrepor ADR; #8: registrar decisões não-óbvias)
- `architecture/flows-wiki/o2p/O2P.md` — canon do framework B2C white-label
- ADRs relevantes do projeto (013, 014, 015, telco-009)

> **Adapte os paths das ADRs** na skill se o seu projeto usar numeração diferente.

---

## Exemplo de output

```
Veredito: GO-CONDICIONAL
Bloqueadores: nenhum crítico
Achados: 10 itens (2 🔴 · 5 🟡 · 3 🟢)

| # | Seção | Trecho Original | 🔧 SF | 📡 TMF | 🏛️ TOGAF | ⚖️ CTO | Sev. | Ação | Owner | ✍️ Você |
|---|-------|----------------|-------|--------|----------|--------|------|------|-------|---------|
| 1 | Notes / linha 64 | "Entry points: ...Partial-Fulfillment..." | ... | ... | 🔴 Contradiz ADR-014 | CONFIRMO | 🔴 | Remover entry-point | TA+SA | |
```

---

## Glossário rápido

| Termo | Significado |
|-------|------------|
| PONR | Point of No Return = fechamento da work order (não a ativação) |
| DRO | Dynamic Revenue Orchestrator — orquestra o O2P, emite UM TMF641 |
| SOM | Service Order Management — executa fora do core |
| SVA | Serviço de Valor Agregado = add-on |
| ETF | Early Termination Fee = multa por rescisão antecipada |
| FulfillmentRequest | Objeto Salesforce que materializa o TMF641 |
| TMF622/641/640 | Product Order / Service Order / Activation APIs |
| TMF637/638 | Product/Service Inventory (assetização) |
| TMF666/678 | Account/Bill Management (billing) |
| TMF676 | Payment Management |
| TMF688 | Event Management |
| TMF646 | Appointment Management |

---

## Licença

MIT — use, adapte e compartilhe.

---

*Criado como parte do método [ADP — Agentic Delivery Platform](https://github.com/p-amers-mx-totalplay-ari/adp) para revisão arquitetural de fluxos Salesforce/TMF.*
