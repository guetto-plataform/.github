# ADR-0013: Desenho da VPC — 3 AZs, `10.20.0.0/16`, seleção de AZ por ID e S3 Gateway Endpoint

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

A camada `10-network` cria a VPC que hospeda nós, pods e load balancers do EKS. O ADR-0003 já definiu subnets
privadas para nós/pods, públicas só para LB e NAT, e um NAT Gateway gerenciado único. Faltava: número de AZs, plano de
endereçamento e como escolher as AZs.

Fatos verificados em 2026-10-13:

- O EKS exige subnets em **pelo menos 2 AZs**; o AWS Load Balancer Controller também
  ([EKS — requisitos de rede](https://docs.aws.amazon.com/eks/latest/userguide/network-reqs.html),
  [best practices](https://docs.aws.amazon.com/eks/latest/best-practices/subnets.html)).
- Com o VPC CNI, **cada pod consome um IP da subnet**.
- Em `us-east-1` o EKS **não aceita** subnets na AZ de ID `use1-az3`. O nome da AZ (`us-east-1a`…) é mapeado para IDs
  de forma diferente em cada conta; nesta conta `use1-az3 = us-east-1e`.
- O módulo `vpc` da biblioteca (v1.0.0) escolhe as primeiras N AZs **por nome** quando `azs` não é informado, e usa
  `10.0.0.0/16` e `map_public_ip_on_launch = true` como defaults.

## Alternativas consideradas

| Alternativa | Prós | Contras | Custo operacional |
|---|---|---|---|
| 2 AZs | Mínimo aceito; menos peças | Menos realista; não exercita distribuição entre 3 zonas | Igual |
| **3 AZs** | Padrão de produção; permite estudar topology spread e afinidade de volumes EBS | Com poucos nós, uma AZ fica sem nó | Igual (subnets e route tables são gratuitas; NAT continua único; tráfego entre AZs até o NAT custa centavos por sessão) |
| CIDR `10.0.0.0/16` (default) | Nenhum ajuste | CIDR de quase todo tutorial: maior chance de sobreposição em peering/VPN/multi-cluster | — |
| **CIDR `10.20.0.0/16`** | Reduz sobreposição futura; força um plano de endereçamento explícito | — | — |
| Subnets /28 dedicadas às ENIs do control plane | Recomendação de best practice contra esgotamento de IPs em upgrades | Módulo não suporta terceiro tipo de subnet; com privadas /19 o risco é desprezível | Médio |
| IPv6 | Elimina escassez de IPv4 | Mais atrito com ferramentas; problema que não temos | Médio |
| S3 Gateway Endpoint como recurso no `guetto-infra` (compondo os outputs do módulo) | ~10 linhas, sem release | Lógica de rede dividida entre módulo e camada | Baixo |
| **S3 Gateway Endpoint no módulo `vpc` da biblioteca (v1.1.0)** | Coeso (é assunto de VPC); reutilizável; primeira release da biblioteca numa mudança de baixo risco, antes das mudanças de EKS | PR + tag + changelog | Baixo |

## Decisão

1. **3 AZs**, escolhidas **por ID** com `exclude_zone_ids = ["use1-az3"]` no data source `aws_availability_zones`
   do `guetto-infra`, e passadas explicitamente ao módulo (`azs`). Garante a portabilidade entre contas (ADR-0003)
   sem alterar a biblioteca.
2. **Plano de endereçamento:**

   | Bloco | Uso | CIDRs |
   |---|---|---|
   | `10.20.0.0/16` | VPC | — |
   | Privadas `/19` × 3 | Nós e pods (~8.190 IPs cada) | `10.20.0.0/19`, `10.20.32.0/19`, `10.20.64.0/19` |
   | Públicas `/24` × 3 | Load balancers e NAT | `10.20.96.0/24`, `10.20.97.0/24`, `10.20.98.0/24` |
   | Reservado | Subnets futuras (bancos, control plane dedicado) | `10.20.99.0` – `10.20.255.255` |

3. `map_public_ip_on_launch = false` nas públicas (nada recebe IP público automático; IPv4 público é cobrado).
4. Tags de descoberta do Load Balancer Controller: `kubernetes.io/role/elb=1` (públicas) e
   `kubernetes.io/role/internal-elb=1` (privadas).
5. **S3 Gateway Endpoint** (gratuito; tira do NAT, cobrado por GB, o tráfego ao S3 — incluindo as camadas de imagens
   do ECR usadas pelos add-ons do EKS) implementado **no módulo `vpc` da biblioteca**: variável
   `enable_s3_gateway_endpoint`, **default `false`** (compatível com consumidores atuais → versão *minor*), associado
   às route tables privadas. Release **v1.1.0**; o `guetto-infra` consome `?ref=v1.1.0` com a variável em `true`.
   A economia neste projeto é pequena (centavos por `up`); o valor é a boa prática sem custo e exercitar o fluxo de
   release da biblioteca (ADR-0010) antes das mudanças de EKS.

## Consequências

- Positivas: portável entre contas; espaço para crescer (Karpenter, prefix delegation, novas subnets) sem
  renumerar; desenho igual ao de ambientes reais.
- Negativas / dívidas assumidas: um único NAT significa tráfego entre AZs até ele e ponto único de falha de saída
  (já aceito no ADR-0003).
- O que faria esta decisão ser revista: conexão com outras redes (verificar sobreposição); necessidade de subnets
  dedicadas ao control plane; adoção de IPv6.

Como o ambiente é efêmero, rever esta decisão custa apenas um `down`/`up`.

## Como validar

- `terraform plan` do `10-network` mostra 3 subnets privadas e 3 públicas em AZs cujos IDs não incluem `use1-az3`.
- Após o `up`, o EKS é criado sem erro de AZ não suportada.
- As route tables privadas têm a rota `pl-xxxx (com.amazonaws.us-east-1.s3) → vpce-xxxx`.

## Referências

- [EKS — requisitos de VPC e subnets](https://docs.aws.amazon.com/eks/latest/userguide/network-reqs.html) — fato documentado
- [EKS Best Practices — VPC e subnets](https://docs.aws.amazon.com/eks/latest/best-practices/subnets.html) — fato documentado / recomendação
- [VPC — Gateway endpoints (sem custo adicional)](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html) — fato documentado
- [ECR — camadas de imagem armazenadas no S3](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html) — fato documentado
