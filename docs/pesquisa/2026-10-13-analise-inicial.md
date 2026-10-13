# Análise inicial (2026-10-13)

Resumo da primeira investigação arquitetural. As decisões derivadas estão nos [ADRs](../adr/).

## Conclusões principais

1. **Organizar por problema, não por tecnologia.** O MVP é um golden path com métricas (ADR-0001).
2. **Correção de premissa:** o Fury é a plataforma do **Mercado Libre**, não da Nubank.
3. **Multi-cluster/hub-spoke no MVP é complexidade sem requisito** (ADR-0004).
4. **CLI/API própria no MVP replica `gh`, `kubectl` e Argo CD.** O software próprio de maior valor é a abstração da aplicação.
5. **Terraform até "cluster + Argo CD"**; o resto é GitOps. Não misturar cluster e recursos K8s no mesmo state.
6. **GitOps não serve para tudo:** operações imperativas, valores de segredo e rollback de emergência precisam de mecanismos explícitos.
7. **Backstage não entra no MVP:** custo de manutenção alto relatado (fontes com viés de fornecedores concorrentes).
8. **Promtail está em EOL desde 02/03/2026** — usar Grafana Alloy.

## Referências

| Tema | Fonte | Tipo |
|---|---|---|
| Maturidade | [CNCF Platform Engineering Maturity Model](https://tag-app-delivery.cncf.io/whitepapers/platform-eng-maturity-model/) | Comunidade |
| Plataforma mínima | [Team Topologies — TVP](https://teamtopologies.com/key-concepts-content/what-is-a-thinnest-viable-platform-tvp) | Autores |
| Evidência crítica | [DORA 2024](https://dora.dev/research/2024/2024-DORA-Report-Infographic.pdf), [TechTarget](https://www.techtarget.com/it-infrastructure/news/366615375/Google-DORA-issues-platform-engineering-caveats) | Pesquisa |
| Anti-padrões | [Octopus](https://octopus.com/devops/platform-engineering/patterns-anti-patterns/), [The New Stack](https://thenewstack.io/9-steps-to-platform-engineering-hell/) | Praticantes |
| Fury | [QCon SF 2024](https://qconsf.com/presentation/nov2024/scaling-innovation-noops-how-mercado-libre-manages-30000-microservices-and-25), [AWS](https://aws.amazon.com/solutions/case-studies/innovators/mercado-libre/) | Empresa |
| Nubank | [CNCF case study](https://www.cncf.io/case-studies/nubank/) | Empresa |
| Argo CD | [Bootstrapping](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/), [ApplicationSet security](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Security/), [Source Hydrator](https://argo-cd.readthedocs.io/en/release-3.5/user-guide/source-hydrator/) | Oficial |
| Multi-cluster | [argocd-agent](https://argocd-agent.readthedocs.io/latest/features/), [Octopus](https://octopus.com/blog/a-comprehensive-overview-of-argo-cd-architectures), [AWS](https://docs.aws.amazon.com/eks/latest/userguide/argocd-concepts.md) | Oficial / fornecedor |
| Ambientes | [Kapelonis — sem branches por ambiente](https://octopus.com/blog/stop-using-branches-deploying-different-gitops-environments) | Praticante |
| Promoção | [Kargo](https://akuity.io/what-is-kargo) | Fornecedor |
| Backstage | [Roadie](https://roadie.io/blog/from-day-0-to-day-2-a-guide-to-planning-and-implementing-backstage/), [dev.to — custo](https://dev.to/jianreis/backstage-how-much-does-it-really-cost-31n5) | Fornecedor / praticante |
| Workload spec | [Score](https://docs.score.dev/docs), [Humanitec ref. arch.](https://humanitec.com/backstage-terraform-ref-arch) | Oficial / fornecedor |
| Terraform | [Anti-padrão provider K8s](https://itnext.io/terraform-dont-use-kubernetes-provider-with-your-cluster-resource-d8ec5319d14a), [OpenTofu x Terraform](https://encore.dev/articles/opentofu-vs-terraform-2026) | Praticante / fornecedor |
| Crossplane | [Crossplane](https://blog.crossplane.io/crossplane-vs-terraform/), [Eficode](https://www.eficode.com/blog/outgrowing-terraform-and-adopting-control-planes) | Fornecedor |
| OTel | [Operator — auto-instrumentação](https://opentelemetry.io/docs/kubernetes/operator/troubleshooting/automatic/) | Oficial |
| Logs | [Promtail EOL](https://grafana.com/docs/loki/latest/clients/promtail/) | Oficial |
| SLOs | [SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) | Oficial |
| Políticas | [VAP x Kyverno x Gatekeeper](https://jorijn.com/en/knowledge-base/kubernetes/security/admission-controllers-kyverno-opa-gatekeeper-cel/), [ADR GOV.UK](https://docs.publishing.service.gov.uk/repos/govuk-infrastructure/architecture/decisions/0023-use-opa-gatekeeper.html) | Praticante |
| Mercado | [CNCF CNPE](https://www.cncf.io/training/certification/cnpe/) | Oficial |
| Custos AWS | [Free Tier 2025](https://aws.amazon.com/about-aws/whats-new/2025/07/aws-free-tier-credits-month-free-plan/), [EKS pricing (secundária)](https://cloudburn.io/blog/amazon-eks-pricing) | Oficial / blog |
