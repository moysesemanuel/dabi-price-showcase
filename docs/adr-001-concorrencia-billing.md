# ADR-001: Modelo de concorrência do billing

**Status:** Aceita (set/2026)

## Contexto

O plano de hardening pedia transações, locking e `SELECT ... FOR UPDATE` no billing. A leitura literal desses itens sugeria um trabalho que **não cabe nesta arquitetura**: o billing acessa o Postgres pelo driver serverless do Neon sobre HTTP, cujas transações recebem uma lista de statements montada de antemão. Não existe transação interativa, então não é possível ler com lock, decidir em código e escrever dentro da mesma transação.

O código não estava desprotegido. Ao longo das PRs de billing foi construído outro modelo, que resolve o mesmo problema com statements únicos. Faltava registrá-lo como decisão, porque cada leitura do plano recriava a mesma dúvida.

## Decisão

Adotar formalmente **exclusão mútua por statement atômico e claims duráveis com lease** como modelo de concorrência do billing.

### Mecanismos

1. **Compare-and-set na linha da fatura.** A atualização só acontece se o status ainda for o esperado (`WHERE id = ? AND status = ?` com `RETURNING`). Webhook, reconciliação e expiração disputam a mesma transição `pendente → paga`, e só a operação vencedora recebe a linha de volta e aplica os efeitos.
2. **Claim durável por fatura, com lease e token de posse.** Adquirido em um único `INSERT ... ON CONFLICT DO UPDATE`, com lease de cinco minutos. Uma fatura paga cujo efeito comercial não terminou continua visível para a reconciliação, que retoma o trabalho interrompido.
3. **Claim de operação por assinatura.** Exclusão mútua entre comandos do usuário, jobs e sincronização de eventos. Quem perde recebe `409` antes de qualquer chamada externa.
4. **Claim do evento de webhook.** Dez entregas simultâneas do mesmo evento geram um único processamento; as demais respondem como duplicata.
5. **Unicidade no banco.** Restrições únicas para eventos e um índice único parcial para pagamentos do provedor.
6. **Idempotency keys** geradas antes da primeira tentativa, para que retries cheguem ao provedor com a mesma chave.

## Opções consideradas

| Opção | Resumo | Veredito |
|---|---|---|
| **A. Manter o modelo atual e documentá-lo** | Nada muda no runtime; statement único é adequado a serverless e já está coberto por testes | ✅ Escolhida |
| B. Migrar o billing para pool via WebSocket | Transações interativas reais, mas troca o driver da camada mais crítica e introduz gestão de conexão em serverless | ❌ Risco alto para atender à letra de um checkbox |
| C. Híbrido, com pool só onde houver invariante multi-linha | Dois drivers na mesma camada é origem clássica de bugs sutis, e não há hoje um caso concreto que exija isso | ❌ Complexidade especulativa |

## Análise

A pergunta que decide não é "queremos `FOR UPDATE`?", e sim "existe hoje um invariante que só uma transação interativa protege?". Todos os pontos críticos do billing são decididos por **uma** linha: a transição da fatura, a posse do claim, o status do evento. Para eles, o statement único é tão forte quanto o lock, e melhor, porque não mantém conexão aberta enquanto o provedor externo responde.

O que o modelo não protege é a escrita concorrente em campos diferentes da mesma assinatura fora do caminho com claim (last-write-wins). Esse problema **não se resolve com `FOR UPDATE`**, e sim com versão de linha (optimistic concurrency), que ficou registrado como o trabalho real a fazer.

## Consequências

- **Mais fácil:** revisar PRs com um critério explícito ("esta operação é decidida por um único statement?") e manter o custo baixo em serverless.
- **Mais difícil:** qualquer invariante que precise abranger várias linhas atomicamente. Se surgir um, este ADR precisa ser revisitado, não contornado.
