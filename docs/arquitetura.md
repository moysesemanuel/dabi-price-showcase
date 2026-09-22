# Arquitetura

## Objetivo

Transformar uma calculadora de preços em um SaaS vendável, mantendo o motor de cálculo como núcleo e adicionando autenticação, multiusuário, persistência, cobrança, suporte, auditoria e uma camada interna de operação da plataforma, preparada para atender novos segmentos sem misturar regras.

## Princípios

- O motor de cálculo é separado da camada visual e da autenticação.
- O **workspace** é a unidade principal de operação do cliente.
- O usuário pertence à plataforma; as permissões pertencem à relação usuário + workspace.
- A camada interna da plataforma nunca vaza para o cliente final.
- Cada segmento (3D, confeitaria, revenda) tem seu próprio modelo de precificação.

## Camadas do produto

| Camada | Responsabilidade |
|---|---|
| Pública | Proposta de valor por segmento, planos, cadastro, login, recuperação de acesso, páginas legais |
| App do cliente | Precificação, histórico, orçamentos, preferências do workspace, membros, assinatura, ajuda e suporte |
| Console interno | Operação da plataforma: usuários, workspaces, assinaturas, pagamentos, eventos, auditoria e saúde operacional |

## Papéis e permissões

**Plataforma:** `super_admin`, `platform_admin`, `support_agent`, `developer`
**Workspace:** `owner`, `manager`, `operator`, `finance`

Os dois eixos são independentes: um mesmo usuário pode ser `super_admin` na plataforma e não ter papel em nenhum workspace, ou ser um usuário comum que é `owner` de um workspace específico.

## Regras de segurança do Super Admin

- Cadastro comum nunca cria `super_admin`, e a interface comum nunca promove alguém a esse papel.
- A última conta `super_admin` não pode ser removida.
- Não existe editor genérico de banco: toda alteração passa por uma ação administrativa específica.
- Entrar no contexto de um workspace é uma ação explícita, com aviso persistente na interface. Não há impersonação silenciosa: as ações continuam atribuídas ao administrador.
- Operações sensíveis registram auditoria com ator, entidade, valor anterior, valor novo, motivo e data/hora.

## Billing

- Catálogo de planos com ciclos mensal e anual.
- Pix para pagamentos avulsos e cartão para recorrência.
- Máquina de estados de assinatura e fatura: checkout, entitlements, paywall, upgrade, downgrade, mudança de ciclo, tolerância, cancelamento e expiração.
- Webhooks assinados, idempotência, jobs agendados e reconciliação com o provedor.

O modelo de concorrência está descrito em [adr-001-concorrencia-billing.md](adr-001-concorrencia-billing.md).

## Integrações

| Serviço | Uso |
|---|---|
| Mercado Pago | Provedor de cobrança |
| Upstash QStash | Jobs externos e reconciliação |
| Mercado Livre | OAuth persistente por workspace |
| ERP DaBi | Produtos e imagens |
| Resend | E-mail transacional |
| Vercel Blob | Upload de arquivos |
| Neon | PostgreSQL serverless |
| Sentry | Monitoramento de erros |
