# ADR-0007: Go e Python como linguagens do projeto

- **Status:** Aceito
- **Data:** 2026-10-13

## Decisão

- **Go:** aplicação de exemplo (API) e, quando justificado por ADR, ferramentas da plataforma
  (CLI de diagnóstico, controller/operator). O ecossistema Kubernetes (client-go, controller-runtime) é Go.
- **Python:** aplicação de exemplo (worker) e automações pontuais.

## Consequência importante

Ter duas linguagens nas apps de exemplo **testa o golden path**: se o chart, a CI e a observabilidade
só funcionarem bem para Go, a plataforma falhou em ser agnóstica.

Nenhuma ferramenta própria é escrita "porque sim": cada uma precisa de um ADR que mostre a dor medida
e por que `gh`, `kubectl`, Argo CD ou Grafana não resolvem.
