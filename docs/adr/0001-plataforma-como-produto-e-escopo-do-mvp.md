# ADR-0001: Tratar a plataforma como produto e manter o MVP enxuto

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 0

## Contexto

O esboço inicial era organizado por tecnologias (Terraform, Argo CD, Go, OpenTelemetry...), não por problemas de
usuário. Esse é o anti-padrão mais citado em Platform Engineering ("build it and they will come").
O DORA 2024 associou o uso **obrigatório** de plataformas internas a quedas de throughput (~8%) e estabilidade (~14%)
— dado correlacional, mas um alerta de design.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| IDP completa (portal, CLI, API, multi-cluster) desde o início | "Vistosa" | Meses de infra sem valor demonstrável; aprendizado raso em cada peça |
| Thinnest Viable Platform guiada por um fluxo de dev | Valor mensurável cedo; cada peça justificada | Menos componentes no início |

## Decisão

Seguir a abordagem *Thinnest Viable Platform*: o MVP é **um golden path** (criar → construir → implantar →
observar → reverter) com métricas mensuráveis. Componentes novos entram somente via ADR que aponte o problema resolvido.
O golden path é caminho preferencial, não obrigatório: deve haver escape hatch documentado.

## Consequências

- Sem portal, CLI própria, API própria, Crossplane ou multi-cluster no MVP.
- Software próprio no MVP limita-se à abstração `guetto-app` (Helm chart) e aos reusable workflows.
- Revisar quando uma dor for **medida** (ex.: passo repetitivo que justifique uma CLI em Go).

## Referências

- [Team Topologies — Thinnest Viable Platform](https://teamtopologies.com/key-concepts-content/what-is-a-thinnest-viable-platform-tvp)
- [CNCF Platform Engineering Maturity Model](https://tag-app-delivery.cncf.io/whitepapers/platform-eng-maturity-model/)
- [DORA 2024 — infográfico](https://dora.dev/research/2024/2024-DORA-Report-Infographic.pdf) · [TechTarget sobre as ressalvas](https://www.techtarget.com/it-infrastructure/news/366615375/Google-DORA-issues-platform-engineering-caveats)
- [Octopus — patterns & anti-patterns](https://octopus.com/devops/platform-engineering/patterns-anti-patterns/)
