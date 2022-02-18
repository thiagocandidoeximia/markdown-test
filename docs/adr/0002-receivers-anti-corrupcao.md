# Receivers comportam-se como um layer de anti-corrupção 

## Status

**Proposed** -> 2022/02/08
### Responsables
* Rafael A. Silva (rafael.amaral@eximia.co)
* Thiago Candido  (thiago.candido@eximia.co)
* William Buzatto (william.buzatto@eximia.co)
* Leandro Almeida (leandro.almeida@cip-bancos.org.br)
* Oscar Azevedo   (oscar.azevedo@cip-bancos.org.br)

## Context

Funcionalidades gerais:
 * Disponibilizar portas de integração.
 * Efetuar todos os tratamentos relacionados a comunicação (Descompactação, Descriptografia, Validação Assinatura, <i>Schema Checking</i>).
 * Viabilizar escalabilidade para balancear o <i>throughput</i>, seguindo regras de <i>throttling</i> definidas pelo o time de negócio associado ao produto que o Receiver esta atendendo.
 * Aguardar que todas <i>Multipart Messages</i>[[1]](#1) cheguem antes de seguir o fluxo.
 * Armazenar Messages[[2]](#2) em tamanhos variáveis.

 Problema: <br />
 Os documentos de integração (Mensagens) poderão variar em formato e tamanho de acordo com o participante e/ou canal de comunicação.

## Decision

Receiver comportara-se como uma camada de anti-corrupção, com o objetivo de:
 1. Reduzir a quantidade de variações de documentos de integração.
 2. Disponibilizar um formato interno padrão em Parquet (TODO: Validar com POC).
 3. Flexibilizar as mudanças contratuais internas.

## Consequences

**Pros**
 * Reduz o acoplamento dos contratos externos (Mensagens no formato do Participante) com os contratos internos (Mensagens que o dominio do produto). Facilitando assim alterações nos contratos internos, sem necessariamente, quebrar os contratos externos.
 * Ao traduzir o arquivo para um formato padrão torna-se possível adicionar componentes de Business Inteligence e Auditória.

**Cons**
 * Deve/Pode adicionar um <i>Penalty</i> no tempo de execução do fluxo completo do negócio.

**Related Quality Attributes**
 * <i>Maintainability (Modifiability)</i>

## Status log
2022/02/08 - Proposed by Eximia.

## Notes and citations
<a id="1">[1]</a> Multipart Messages são mensagens que foram quebradas ou páginadas. <br />
<a id="2">[2]</a> Messages neste contexto está associado a qualquer channel (API, Message Broker ou File System).