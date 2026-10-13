# Documentação da Guetto Platform

Documentação central do projeto: decisões, arquitetura, operação e diário de aprendizado.

> Estado: **esboço**. A estrutura existe para receber conteúdo à medida que as decisões forem tomadas.
> Nenhum documento deve ser escrito "antes da hora" — cada pasta indica em que fase passa a ser preenchida.

## Mapa

| Pasta | Conteúdo | Quando preencher |
|---|---|---|
| [visao/](visao/) | Problema, personas, objetivos e métricas de sucesso | Fase 0 |
| [adr/](adr/) | Architecture Decision Records (o "porquê" de cada escolha) | Sempre que uma decisão for tomada |
| [arquitetura/](arquitetura/) | Diagramas C4 (contexto, containers), implantação, custos | Fase 0 → evolui em cada fase |
| [golden-paths/](golden-paths/) | Definição dos caminhos padronizados para devs | Fase 3 |
| [operacao/](operacao/) | Instalação (up/down), SLOs, runbooks | Fase 2 (up/down), Fase 4 (SLOs/runbooks) |
| [seguranca/](seguranca/) | Modelo de ameaças, IAM, segredos, políticas | Fase 5 (IAM/segredos desde a Fase 1) |
| [aprendizado/](aprendizado/) | Diário de estudo: conceitos, erros, experimentos | Contínuo |
| [pesquisa/](pesquisa/) | Análises e referências que embasam as decisões | Antes de cada decisão relevante |
| [roadmap.md](roadmap.md) | Fases, critérios de conclusão e status | Atualizar ao fim de cada fase |

## Convenções

- Idioma: português. Termos técnicos consagrados permanecem em inglês (GitOps, rollback, golden path).
- Toda afirmação sobre ferramentas deve ter link para a fonte. Diferencie **fato documentado** de **avaliação/opinião**.
- ADRs são imutáveis depois de aceitos: para mudar uma decisão, crie um novo ADR que **substitui** o anterior.
- Diagramas em texto (ASCII ou Mermaid) para serem versionáveis e revisáveis em PR.
