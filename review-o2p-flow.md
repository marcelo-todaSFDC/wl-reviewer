---
name: review-o2p-flow
description: >
  Revisão arquitetural de um artefato O2P (flows-wiki) por 4 agentes especialistas em
  paralelo: Arq. Técnico Salesforce Comms Cloud, Arq. Funcional TM Forum / eTOM / ODA,
  Arq. Estratégico TOGAF, e Validação Cross / CTO. Produz uma tabela de achados com
  Seção, Trecho Original, parecer de cada agente, veredito CTO, severidade e ação sugerida.
  Use quando: revisar qualquer .md em architecture/flows-wiki/ antes de promover para curated.
  Invoke: review-o2p-flow <caminho-relativo-ao-md>
---

# Skill: review-o2p-flow

## Propósito

Executar revisão arquitetural estruturada de um artefato O2P antes da promoção para `status: curated`. Replica o método de 4 agentes usados na revisão inaugural (Order-Cancellation + MACD-Disconnect, agosto/2026) como processo repeatable e versionado.

## Pré-condições (leia antes de executar)

Antes de invocar os agentes, leia obrigatoriamente:

1. `SOUL.md` — non-negotiables #7 e #8 (nunca sobrepor ADR em silêncio; registrar decisões não-óbvias)
2. `architecture/flows-wiki/o2p/O2P.md` — canon do framework B2C white-label
3. `architecture/decisions/013-order-closure-sequence-activate-ponr-billing.md` — sequência activate→PONR→billing; escopo: Alta e Cambio de Plan; bajas = open point (§9·3, §9·6)
4. `architecture/decisions/014-dro-governed-fallout-two-lanes-by-ponr.md` — all-or-nothing; Partial-Fulfillment desabilitado no TO-BE Totalplay
5. `architecture/decisions/015-core-master-of-commercial-asset-lifecycle.md` — Core SF é master do Asset comercial
6. `architecture/decisions/telco-009-order-configuration-efficiency-lean-decomposition.md` — Multi-Site OFF; B2C single-site
7. O próprio MD alvo — leia na íntegra antes de despachar os agentes

## Execução

### Passo 1 — Ler o artefato alvo

Leia o MD informado. Extraia:
- Frontmatter: `journey`, `step`, `movement`, `status`
- Seções principais: Flow (Mermaid), Steps (tabela), Notes
- Referências a outros fluxos (`[[...]]`), ADRs e TMF APIs citadas

### Passo 2 — Despachar 3 agentes especialistas em paralelo

Despache os 3 agentes simultaneamente via Agent tool. Cada um recebe o conteúdo completo do MD + os documentos de pré-condição relevantes ao seu domínio.

---

#### Agente 1 — Arq. Técnico Salesforce Comms Cloud (DRO/SOM)

**Persona:** Arquiteto técnico sênior de Salesforce Communications Cloud, especialista em DRO (Dynamic Revenue Orchestrator), SOM (Service Order Management), Comms Cloud OM, Apex e data model da plataforma.

**Missão:** Revisar o artefato sob a perspectiva de implementabilidade em Salesforce Comms Cloud:
- Os SF Objects citados existem e são os corretos? (ex.: `FulfillmentRequest` ≠ `Service Order`)
- Os passos do DRO são implementáveis como orchestration items / compensating actions?
- Os estados do fluxo são testáveis (unit + integração)?
- Há lógica de decomposição não-especificada que o DRO precisaria mas o artefato omite?
- Há eventos TMF688 de retorno e timeout guards para estados assíncronos?
- Consistência com ADR-013 (sequência activate→close→billing), ADR-014 (all-or-nothing), ADR-015 (Asset master)?

**Output esperado:** lista de achados no formato:
```
SEÇÃO | TRECHO ORIGINAL | OBSERVAÇÃO | SEVERIDADE (🔴/🟡/🟢/⚪)
```

---

#### Agente 2 — Arq. Funcional TM Forum / eTOM / ODA

**Persona:** Arquiteto funcional de telecomunicações, especialista em TM Forum (TMF Open APIs), eTOM v2, ODA (Open Digital Architecture) e modelos SID/Information Framework.

**Missão:** Revisar o artefato sob a perspectiva de conformidade com padrões TM Forum:
- O mapeamento eTOM no header está correto e completo? (verificar todos os processos L2 cobertos, não só o principal)
- As TMF APIs citadas são as corretas para cada operação? (ex.: TMF676 = Payment, não Rating; TMF678 = Bill Management; TMF641 = Service Order; TMF639/652 = Resource)
- Os estados TMF622 citados são canônicos? Ou são customizados sem declaração?
- O fluxo modela corretamente o ciclo `orderItem.action` (add/delete/modify)?
- Sub-estados internos do DRO estão claramente distinguidos de estados canônicos TMF?
- O service order TMF641 emitido pelo DRO tem seu estado refletido corretamente no ciclo de vida?
- Princípios ODA respeitados: vendor-neutral, event-driven, separação de camadas?

**Output esperado:** lista de achados no mesmo formato acima.

---

#### Agente 3 — Arq. Estratégico TOGAF / Governança

**Persona:** Arquiteto estratégico, especialista em TOGAF ADM (4 domínios: Business/Data/Application/Technology), Architecture Repository, rastreabilidade de requisitos e governança de ADRs.

**Missão:** Revisar o artefato sob a perspectiva de governança e completude arquitetural:
- Decisões não-triviais assumidas sem ADR? (SOUL #7 e #8)
- Referências a ADRs existentes nas notas? (rastreabilidade)
- Contradições com ADRs vigentes? (especialmente ADR-013 §9·3/§9·6 para PONR de bajas)
- Cobertura dos 4 domínios TOGAF no artefato?
- Vocabulário consistente com o canon? Termos ambíguos ou sobrecarregados?
- Concerns de segurança, retenção de dados ou compliance ausentes? (verificar HIGH-LEVEL-SECURITY se o fluxo toca visibilidade de objetos ou fechamento de conta)
- Artefatos cruzados (T2C.md, R2C.md) em conflito com o que este fluxo afirma?

**Output esperado:** lista de achados no mesmo formato acima.

---

### Passo 3 — Agente Cross / Validação CTO

Após receber os outputs dos 3 agentes, despache um **4º agente** com todos os achados consolidados.

**Persona:** CTO / Arquiteto Cross — visão sistêmica, sem viés de domínio. Papel: árbitro e sintetizador.

**Missão:**
1. Para cada achado dos 3 agentes: confirmar, moderar ou refutar — com justificativa baseada nos documentos de pré-condição (não em opinião)
2. Elevar severity se 2+ agentes convergem no mesmo problema
3. Rebaixar severity se o achado conflita com o que os ADRs estabelecem (ex.: agente confunde activation com PONR — refutar citando ADR-013)
4. Identificar achados cruzados entre agentes que se complementam
5. Produzir veredito final: **GO**, **GO-CONDICIONAL** ou **NO-GO** com os bloqueadores listados

**Output esperado:**
- Tabela consolidada: `# | Seção | Trecho Original | Arq.SF | Arq.TMF | Arq.TOGAF | Veredito CTO | Sev. Final | Ação Sugerida | Owner`
- Veredito geral com justificativa

---

### Passo 4 — Produzir output final

Com a tabela consolidada do Agente 4:

1. **Exibir o resumo** ao usuário: veredito, bloqueadores críticos, pontos fortes
2. **Gerar um arquivo Excel** (`.xlsx`) com openpyxl na mesma pasta do artefato revisado:
   - Aba única com o nome do artefato (ex.: `O2P-Order-Cancellation`)
   - Linha 1: título com o nome do artefato e o veredito
   - Linha 2: path do arquivo revisado
   - Linha 4: cabeçalho — `# | Seção / Linha | Trecho Original | 🔧 Arq. SF | 📡 Arq. TMF | 🏛️ Arq. TOGAF | ⚖️ Veredito CTO | Sev. | Ação sugerida | Owner | ✍️ Parecer [Arquiteto]`
   - Linhas seguintes: um row por achado
   - Coluna `✍️ Parecer` sempre vazia — para o arquiteto humano preencher
   - Coluna `Sev.` com cor de fundo: 🔴 = vermelho claro, 🟡 = amarelo claro, 🟢 = verde claro
   - Colunas de texto com wrap e largura adequada
   - Nome do arquivo: `Revisao-<basename-do-md>.xlsx`
3. Informar ao usuário o path do arquivo gerado e abri-lo se possível

## Glossário de referência rápida

| Termo | Significado |
|-------|------------|
| PONR | Point of No Return = fechamento da work order (não a ativação) |
| DRO | Dynamic Revenue Orchestrator — orquestra o O2P, emite UM TMF641 |
| SOM | Service Order Management — executa fora do core |
| SVA | Serviço de Valor Agregado = add-on (ex.: TV extra, antivírus) |
| ONT/CPE | Optical Network Terminal / Customer Premises Equipment = equipamento físico no cliente |
| ETF | Early Termination Fee = multa por rescisão antecipada |
| TMF622 | Product Order Management API |
| TMF641 | Service Order Management API — emitido pelo DRO |
| TMF639/652 | Resource Inventory / Resource Order — externo, SOM-interno |
| TMF640 | Service Activation API |
| TMF637/638 | Product Inventory / Service Inventory — assetização |
| TMF666/678 | Account Management / Customer Bill Management — billing |
| TMF676 | Payment Management — captura de pagamento e refund |
| TMF688 | Event Management — notificações assíncronas |
| TMF646 | Appointment Management — agendamento de visita técnica |
| eTOM 1.3.3 | Order Handling (processo L2 principal) |
| eTOM 1.4.5 | Service Config & Activation (delegado para deactivate/recover) |
| Cancel-Replace | Modelo Comms Cloud OM onde nova ordem supersede a original |
| FulfillmentRequest | Objeto SF que materializa o TMF641 (Service Order) |
