# Estratégia de testes

## Números

- **58 suítes de testes unitários**, rodando com o test runner nativo do Node.js
- **9 suítes de integração** contra PostgreSQL real
- Executadas em **toda pull request** pelo GitHub Actions

## O que é coberto

| Área | Exemplos |
|---|---|
| Precificação | Motor de cálculo, modelos por segmento (3D, confeitaria, revenda), margem por canal |
| Billing | Máquina de estados, checkout, upgrade com proração, downgrade, mudança de ciclo, entitlements, notificações, reconciliação |
| Concorrência | Claims de fatura e de assinatura, processamento concorrente de webhooks, concorrência na autenticação |
| Provedor de pagamento | Integração com Mercado Pago, rota de webhook, autenticação dos jobs agendados |
| Acesso e segurança | Controle de acesso por papel, proteção de rotas, redirecionamento de login, rate limit, consentimento legal |
| Operação | Observabilidade por rota, throttling de erros, configuração do Sentry, migrações de banco |
| Produto | Estrutura das landings, página de planos, consistência de nomes de planos, upload de imagens |

## Por que testes de integração com banco real

As garantias de concorrência do billing dependem do comportamento do Postgres: `ON CONFLICT`, `RETURNING`, índices únicos parciais. Um mock não prova que duas operações simultâneas resultam em um único vencedor, então esses cenários rodam contra um banco de verdade.

## Pipeline de CI

```yaml
name: check

on:
  pull_request:
  push:
    branches: [main, release/homologation]

concurrency:
  group: check-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  check:
    name: lint + typecheck + testes + build
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - name: Validação completa
        run: npm run check   # lint → typecheck → testes → build
      - name: Espaços em branco no diff
        if: github.event_name == 'pull_request'
        run: git diff --check "${{ github.event.pull_request.base.sha }}" "${{ github.sha }}"
```
