# Receivers devem armazenar as mensagens no S3 

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
 As mensagens chegam em diversos tamanhos (4MB - 500MB).



## Decision

As mensagens recebidas serão armazenadas no S3 Storage. 

## Consequences

**Pros**
 * Viabiliza o uso de ferramentas de Orquestração lidar com arquivos medios(4mb>) ou grandes(40mb >).
 * Viabiliza a ferramenta de auditória funcionar em um fluxo separado ao de integração.

**Cons**
 * Aumenta o acoplamento com o S3 e a estrutura de arquivos.
 * Custo com armazenamento e necessidade de estratégia de retenção.

**Related Quality Attributes**
 * <i>Performance and Efficiency (Capacity and Time behaviour)</i>


## Status log
2022/02/08 - Proposed by Eximia.

## Notes and citations
<a id="1">[1]</a> Multipart Messages são mensagens que foram quebradas ou páginadas. <br />
<a id="2">[2]</a> Messages neste contexto está associado a qualquer channel (API, Message Broker ou File System).