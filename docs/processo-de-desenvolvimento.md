# Processo de desenvolvimento

## Especificação antes do código

O desenvolvimento segue um fluxo orientado a especificação (Spec-Driven Development):

1. **Escopo oficial congelado por ciclo.** Nenhuma funcionalidade nova entra sem atualização explícita do documento de escopo e do plano.
2. **Plano com 24 fases**, cada uma com objetivo, ações, critério de aceite e registro de execução. As fases vão de infraestrutura e segurança até billing ponta a ponta, Super Admin, observabilidade, testes finais, backup, smoke test em produção e auditoria final.
3. **Definição explícita de "pronto".** O produto só é considerado 100% quando está funcional para clientes reais, com cobrança homologada, seguro, operável sem acesso manual ao banco, monitorado, testado e documentado.
4. **ADRs para decisões estruturais**, registrando contexto, opções, trade-offs e consequências.

## IA como parte do processo

Trabalho com agentes de IA (Claude Code) sobre essas especificações, não no improviso:

- **Análise e planejamento:** levantamento do estado do código, desenho de arquitetura e escrita dos planos e ADRs.
- **Implementação:** em branch, com revisão commit a commit antes do merge.
- **Verificação:** o CI (lint, typecheck, testes, build) valida tudo o que é gerado. A IA acelera, e os testes e a revisão garantem a correção.
- **Contexto para os agentes:** arquivos de instrução no repositório orientam as ferramentas sobre as convenções do projeto.

## Registro de evidência

Toda afirmação de status no plano precisa ser verificável no código ou trazer data e ambiente em que foi observada, e é registrada na branch principal por PR.

A regra nasceu de um problema real: uma homologação de cartão recorrente foi registrada apenas na branch de homologação e, durante doze dias, a fase apareceu como não iniciada na `main`, a ponto de uma avaliação de prontidão apontar como pendente um trabalho já feito. Depois de uma auditoria commit a commit entre as branches, o processo passou a exigir o registro por PR, e ganhou um comando para detectar divergências de checklist entre as duas.

## Ambientes

| Ambiente | Uso |
|---|---|
| Local | Desenvolvimento, com fallback sem banco |
| Preview | Um deploy por PR na Vercel |
| Homologação | Branch dedicada, com integrações em modo de teste |
| Produção | Branch principal |
