# Receivers deve possuir um projeto referência. 

## Status

**Proposed** -> 2022/02/08
### Responsables
* Rafael A. Silva (rafael.amaral@eximia.co)
* Thiago Candido  (thiago.candido@eximia.co)
* William Buzatto (william.buzatto@eximia.co)
* Leandro Almeida (leandro.almeida@cip-bancos.org.br)
* Oscar Azevedo   (oscar.azevedo@cip-bancos.org.br)

## Context

Receivers possuem passos que são obrigatórios e não é variável pelo produto.

Lista de passos:
1. Descompactar <i>payload</i>.
2. Descriptograr <i>payload</i>.
3. Validação do schema do <i>payload</i>.
4. Validação de formulário (Comportamento de validação é determinado pelo o time de negócio a partir de extensão).
5. Identificar participante (<i>Multi-tenant</i>).
6. Agrupar <i>Multi-part messages</i>.
7. Traduzir para documento interno de integração (Anti-corrupção).

Problema: <br />
Pode-se tornar uma dor garantir que todos os passos obrigatórios sejam respeitados. Não podemos permitir o time de negócio negligenciar os passos obrigatórios (Somente em situação deliberadas e argumentos de negócio).

## Decision

Um projeto referência será desenvolvido pelo o time especializado nas funcionalidades obrigatórias e disponibilizado em repositório GIT. Assim os times de negócios podem efetuar um Fork e usar o código como base. 
Também entendemos que é necessário o acompanhamento próximo do <i>tech leader</i> em caso de mudanças destes passos obrigatório e qualquer mudança nestes passos devem ser aprovados a nível de negócio e com acompanhamento do time de Arquitetura e/ou Arquiteto Responsável.

## Consequences

**Pros**
 * Reduz as variações nas soluções possíveis de um Receiver.
 * Reforça os comportamentos padrões mas reduzindo as barreiras nas alterações do time de negócio em situações excepcionais.

**Cons**
 * Code Review torna-se obrigatório.
 * Explicitar e documentar os passos obrigatórios é fundamental para evitar mudanças indesejadas no futuro.

**Related Quality Attributes**
 * <i>Maintainability (Reusability)</i>

## Status log
2022/02/08 - Proposed by Eximia.