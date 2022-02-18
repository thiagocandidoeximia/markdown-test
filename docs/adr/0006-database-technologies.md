# Database technologies

## Status

**Proposed** -> 2022/02/08
### Responsibles
* Rafael A. Silva (rafael.amaral@eximia.co)
* Thiago Candido  (thiago.candido@eximia.co)
* William Buzatto (william.buzatto@eximia.co)
* Leandro Almeida (leandro.almeida@cip-bancos.org.br)
* Oscar Azevedo   (oscar.azevedo@cip-bancos.org.br)

## Context

A Arquitetura é inspiradas em dois modelos arquiteturais: 
1. <i>Pipes and Filters[[1]](#1)</i>
2. <i>Architecture Distributed in Services EDA[[2]](#2)[[3]]</i>

Os componentes de integração estão regidos pela <i>Pipes and Filters</i> e a camada de negócio e gerencial esta totalmente relacionado a arquitetura de serviços com eventos. 

Situação atual: <br />
Atualmente o paradigma e a tecnologia de banco de dados go-to são respectivamente relacional e PostgreSQL, para todas as bases transacionais das aplicações. Com a proposta da nova arquitetura, também existe a possibilidade do uso não só de outras tecnologias mas, especificamente, de diferentes paradigmas, quando se fizer necessário. As tecnologias propostas são integralmente serviços oferecidos pela AWS, devido auto-gerenciabilidade de tais serviços e de seu custo benefício em relação a escala. 

Problema: <br />
Variar as tecnologias de banco de dados sem encontrar um argumento de negócio para justificar somente adiciona custo. 

## Decision

### **1. Relacional**

A principal opção de tecnologia para o paradigma relacional é o Amazon RDS for PostgreSQL, por se tratar da versão gerenciada da AWS da mesma tecnologia já utilizada nos sistemas CIP. Facilitando a configuração, a operação e a escalabilidade da estrutura de banco de dados, o AWS RDS for PostgreSQL entrega benefícios como:
- Backup e recuperação
- Alta disponibilidade e réplicas de leitura
- Monitoramento e métricas
- Isolamento e segurança

Também, atualmente, é compatível com PostgreSQL 9.6, 10, 11, 12 e 13.

#### **1.1 Quando utilizar?**
- Integridade e consistência são essenciais (ACID)
- Queries complexas e altamente relacionadas
- Não há mudanças frequentes nos schemas
- "Scale up" previsto é suficiente 

###  **2. NoSQL - Document**
Para workloads NoSQL, a primeira opção é um banco de documentos. Aqui, propomos a utilização do AWS DocumentDB, que é um serviço de banco de dados escalável, compatível com o MongoDB, que entrega benefícios como:
- Automatização de provisionamento de hardware
- Nove 9s de durabilidade 
- Replicação automática
- Backup contínuo
- Isolamento de rede

Drivers e as ferramentas existentes com as APIs MongoDB 3.6 e 4.0 são compatíveis.

#### **2.1 Quando utilizar?**
- Escalabilidade horizontal é essencial
- Queries baseadas em documentos compostos
- Schemas dinâmicos e flexíveis

###  **3. NoSQL - Key Value**
Bancos chave e valor podem ser utilizados para atender requisitos de caching por sua escala, performance e baixo custo. O Amazon ElastiCache é um serviço gerenciado, compatível com Redis ou Memcached, focado em armazenamento em cache. Alguns casos de uso são:
- Acelerar performance das aplicações
- Reduzir pressão sobre o banco de dados do back-end
- Armazenar conjuntos de dados não duráveis

#### **3.1 Quando utilizar?**
- Armazenamento e entrega de dados não duráveis, como sessões de usuário
- Obter dados frequentemente acessados com latência baixa
- Atender key-value queries

## Consequences

**Pros**
- Opções de tecnologias quem, em conjunto, oferecem ótimas soluções para diferentes problemas
- Serviços gerenciados não trazem grande overhead operacional
- Serviços são compatíveis com tecnologias amplamente documentadas e com forte comunidade ativa (MongoDB, PostgreSQL, Redis, Memcached)

**Cons**
- O time de desenvolvimento não possui experiência nas tecnologias NoSQL
- Os schemas, por motivos históricos, são totalmente baseados no paradigma relacional. Alterar isso pode ser de grande impacto, além de necessitar de um grande esforço por parte do time.
- Schemas não parecem sofrer mudanças frequentemente, e devido a isso, a argumentação para escolha do paradigma NoSQL pode ser enfraquecido


**Related Quality Attributes**
 * <i>Tech Radar - Selecionar tecnologias com maior Fit dentro do time é maior que tecnologia novas com beneficios mínimos.</i>

## Notes and Citations
<a id="1">[1]</a> <a href="https://docs.microsoft.com/pt-br/azure/architecture/patterns/pipes-and-filters">Pipes and filters:</a> Decompor uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados. Isso pode melhorar o desempenho, escalabilidade e reutilização, permitindo que os elementos de tarefa que executam o processamento sejam implantados e escalados de forma independente. <br />

<a id="2">[2]</a> <a href="https://www.gartner.com/en/information-technology/glossary/eda-event-driven-architecture">Event Driven Architecture</a><br />

<a id="3">[3]</a> <a href="http://www.psinaptic.com/link_files/distributed_computing.pdf">Entendendo os principais desafios da computação distribuída.</a><br />

<a id="4">[4]</a> <a href="https://aws.amazon.com/pt/documentdb/">Amazon DocumentDB</a><br />

<a id="5">[5]</a> <a href="https://aws.amazon.com/pt/elasticache/">Amazon ElastiCache</a><br />

<a id="6">[6]</a> <a href="https://aws.amazon.com/pt/rds/postgresql/">Amazon RDS for PostgreSQL</a><br />