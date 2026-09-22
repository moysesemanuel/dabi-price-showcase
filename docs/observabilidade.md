# Observabilidade com Sentry

## Objetivo

Permitir investigar falhas de produção a partir de eventos reais, sem depender só de reprodução local e logs, e **sem que dados pessoais ou de pagamento saiam da aplicação**.

## Escopo

- Captura de erros nos runtimes de browser, Node.js e Edge do Next.js.
- Captura global de erros de requisição, sem alterar rotas de negócio individualmente.
- Ambientes separados no mesmo projeto: `production`, `preview` e `hml` (homologação). Desenvolvimento local fica desligado por padrão para evitar ruído.
- Upload de source maps no deploy, para stack traces legíveis.

## Privacidade como requisito

- Um filtro `beforeSend` remove integralmente os dados de requisição e de usuário antes do envio.
- Valores sensíveis são mascarados de forma recursiva em exceções, stack traces e contexto.
- Corpo de requisição, cabeçalhos de autorização, cookies, query strings, identidade do usuário, tokens e payloads de pagamento **não chegam ao Sentry**.
- Os eventos carregam apenas o request ID, que permite correlacionar com os logs estruturados da própria aplicação.

## Segurança da configuração

- A inicialização é condicional à presença do DSN: sem configuração de monitoramento, a aplicação continua funcionando normalmente.
- O token de upload de source maps é um segredo exclusivo do build, com escopo mínimo, e nunca é exposto ao cliente, a logs ou ao repositório.
- O upload só roda quando o build acontece na Vercel com todas as credenciais presentes, o que impede o uso acidental de tokens locais.

## Fora de escopo nesta etapa

Session Replay, tracing de performance e profiling ficaram de fora de propósito, para começar pelo que traz mais valor com menor risco de coletar dados demais.
