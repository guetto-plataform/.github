# Custos

> Estado: **estimativa não verificada**. Substituir pelos valores reais do Cost Explorer após as primeiras sessões.
> Preços variam por região; conferir sempre na página oficial de preços da AWS.

## Orçamento

- Créditos disponíveis: **US$ 200**, conta no **Paid plan**, região **us-east-1**.
- **Os créditos expiram em 30/12/2026** (console → Billing → Credits, conferido em 2026-10-13). A suposição anterior de
  "12 meses a partir do cadastro" não vale para esta conta: o FAQ da AWS é inconsistente entre 6 e 12 meses
  ([FAQ](https://aws.amazon.com/free/free-tier-faqs/)), e a data do console é a fonte de verdade.
  A API (`aws freetier get-account-plan-state`) informa o saldo, mas não a validade, em conta Paid.
- No Paid plan não há bloqueio de serviços, mas **o que exceder os créditos é cobrado no cartão** — as proteções abaixo
  deixam de ser opcionais. **A partir de 31/12/2026 todo uso é cobrado no cartão.**
- Novos créditos do Free Tier são só para quem nunca teve conta AWS ([FAQ](https://aws.amazon.com/free/free-tier-faqs/));
  não contar com "trocar de conta" para renová-los (ADR-0011).

## Componentes que cobram por hora (estimativa)

| Item | Ordem de grandeza | Observação |
|---|---|---|
| Control plane EKS | ~US$ 0,10/h | Cobrado desde a criação, mesmo sem workloads. Versões em *extended support* custam ~6x mais ([fonte secundária](https://cloudburn.io/blog/amazon-eks-pricing)) |
| Nós EC2 (ex.: 2× t3.medium / t4g.medium) | ~US$ 0,07–0,09/h | Spot pode reduzir bastante; a avaliar |
| Load balancer | ~US$ 0,02–0,03/h + uso | Preferir **um** LB compartilhado via Ingress |
| NAT Gateway (1, gerenciado) | ~US$ 0,045/h + por GB processado | Escolhido por realismo (ADR-0003). Pull de imagens passa por ele |
| IPv4 público | cobrança por IP/hora | Minimizar IPs públicos |
| Secrets Manager | por segredo/mês | SSM Parameter Store (standard) é gratuito |

**Ordem de grandeza por sessão:** ~US$ 0,25–0,40/h → uma sessão de 4h ≈ US$ 1–2.
**O risco real** não é a sessão, é esquecer o ambiente ligado: ~US$ 0,30/h × 24h × 30 dias ≈ US$ 200 (todo o crédito).

## Proteções obrigatórias

- [x] AWS Budget de custo **bruto acumulado** (créditos excluídos), avisos em US$ 75 e US$ 150 — criado no bootstrap
  (ADR-0011). Período `CUSTOM` até 30/12/2026: **a AWS apaga o budget após essa data**.
- [ ] Antes de 30/12/2026: substituir por um budget **mensal de custo líquido** com limite baixo (o custo passa a ir
  para o cartão).
- [ ] Workflow agendado ("reaper") que executa `make down` todo dia num horário fixo.
- [ ] Tags padrão (`project=guetto`, `ephemeral=true`) em todos os recursos para auditoria.
- [ ] Script de verificação pós-`down` que lista recursos órfãos (LBs, ENIs, volumes EBS).

## Registro de medições

| Data | Duração | Custo real | Observações |
|---|---|---|---|
| | | | |
