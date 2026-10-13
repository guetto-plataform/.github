# ADR-0014: Reaper diário via GitHub Actions às 23h (Brasília), com execução de segurança

- **Status:** Aceito
- **Data:** 2026-10-13
- **Fase do roadmap:** 1

## Contexto

O ADR-0002 tornou obrigatório um reaper: o risco de custo é esquecer o ambiente ligado (~US$ 0,30/h ≈ US$ 7/dia),
e o budget (ADR-0011) avisa tarde por design. Falta definir onde o reaper roda e com que política.

Fatos verificados em 2026-10-13 sobre o `schedule` do GitHub Actions
([eventos](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows),
[sintaxe](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)):

- Execuções podem **atrasar** e, sob carga alta, **ser descartadas** — é *best effort*.
- Em repositório público, workflows agendados são **desativados após 60 dias** sem atividade.
- Notificações de falha vão para quem editou por último a linha do cron.
- Aceita fuso IANA (`timezone`); carga maior na virada da hora.

## Alternativas consideradas

| Alternativa | Prós | Contras | Custo operacional |
|---|---|---|---|
| **GitHub Actions agendado** | Reusa o mesmo `scripts/down.sh` e a role OIDC; nenhuma infra persistente nova | *Best effort* (atraso/descarte) | Zero |
| EventBridge Scheduler + CodeBuild/Lambda | Confiável | Infra persistente nova no bootstrap; outro caminho de execução ou drift com o Terraform | Baixo, mas mais peças |
| TTL verificado de hora em hora | Pior caso N+1 h; tolera descartes | Precisa registrar quando o ambiente subiu | Baixo |
| **Horário fixo diário** | Simples; previsível | Pior caso ~24 h (ou ~48 h se a execução for descartada); derruba sessões após o horário | Zero |
| Variável para pular o reaper numa data | Permite estudar após o horário | Mais um mecanismo; o autor raramente estuda à noite | Baixo |

## Decisão

1. Reaper como **workflow agendado do GitHub Actions** que executa `scripts/down.sh` com a role OIDC do environment
   da conta (ADR-0012).
2. **Horário fixo: todos os dias às 23:00** com `timezone: "America/Sao_Paulo"`.
3. **Execução de segurança às 02:17** (minuto quebrado, fora do pico da virada da hora). Se a das 23:00 rodou, esta
   encontra tudo derrubado e não faz nada — o `down` precisa ser **idempotente**.
4. **Sem mecanismo para pular** o reaper: se ele derrubar uma sessão noturna, basta um novo `up`.
5. `concurrency` compartilhado entre os workflows de up, down e reaper da mesma conta.
6. Rede de segurança: o budget (ADR-0011). Freio de emergência nativo na AWS fica como evolução, só se o reaper
   falhar de fato.

## Consequências

- Positivas: o caminho do `down` é exercitado todo dia; nenhuma infraestrutura nova.
- Negativas / dívidas assumidas:
  - Pior caso de ambiente esquecido: ~24 h (~US$ 7); com as duas execuções descartadas, ~48 h (~US$ 14).
  - Sessões após 23:00 são derrubadas (aceito: raras).
  - Após 60 dias sem atividade no repositório, o agendamento é desativado; reativar ao retomar (runbook).
- O que faria esta decisão ser revista: reaper falhar ou ser descartado de fato; custo por hora do ambiente
  aumentar; estudo noturno frequente (rever TTL).

## Como validar

- Subir o ambiente, aguardar as 23:00 e verificar que o workflow derrubou tudo (sem órfãos).
- A execução das 02:17 termina com sucesso sem alterações.
- Disparo manual do reaper com o ambiente já derrubado: sucesso, sem mudanças.

## Referências

- [GitHub — eventos de workflow (`schedule`)](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) — fato documentado
- [GitHub — sintaxe (`timezone`)](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) — fato documentado
