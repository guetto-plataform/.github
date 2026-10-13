# ADR-0008: GHCR como registry de imagens

- **Status:** Proposto
- **Data:** 2026-10-13

## Contexto

A conta AWS é efêmera e intercambiável. Um registry dentro da conta (ECR) seria destruído ou precisaria ser migrado.

## Alternativas

| Alternativa | Prós | Contras |
|---|---|---|
| **GHCR** | Fora da AWS: sobrevive a `down` e à troca de conta; integrado ao GitHub Actions | Pull pela internet (sem NAT, os nós precisam de saída) |
| ECR | Nativo, IAM integrado, scan | Atrelado à conta; precisa ficar fora da camada efêmera ou ser migrado |

## Decisão (proposta)

GHCR. Imagens públicas ou pull secret via External Secrets se privadas.
