# Messages Broker

## Status

**Proposed** -> 2022/02/08

### Responsables

- Rafael A. Silva (rafael.amaral@eximia.co)
- Thiago Candido (thiago.candido@eximia.co)
- William Buzatto (william.buzatto@eximia.co)
- Leandro Almeida (leandro.almeida@cip-bancos.org.br)
- Oscar Azevedo (oscar.azevedo@cip-bancos.org.br)

## Context

<i>Architecture Design: Pipes and Filters[[1]](#1)</i>

Problema: <br />
Em arquiteturas distribuídas e assíncronas é preciso garantir que as comunicações estão ocorrendo da forma correta. Mesmo em situação que algum componente da arquitetura esteja fora do ar momentâneamente. Também é necessário um Broker que seja capaz de suportar grande quantidade de mensagens e um <i>throughput</i> alto.

## Decision

### Aspectos avaliados:

| Aspectos             | Kafka                                                                                                                                   | AWS Kinesis                                                                                 | AWS SQS/SNS                   |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ----------------------------- |
| Messaging guarantees | At least once per normal connector. <br /> Precisely once with Spark direct Connector.                                                  | At least once unless you build deduping <br /> or idempotency into the consumers            | At least once                 |
| Ordering guarantees  | Guaranteed within a partition.                                                                                                          | Guaranteed within a shard.                                                                  | Guaranteed within a FIFO SQS. |
| Throughput           | ~30000/s[[2]](#2)                                                                                                                         | a bit slower than Kafka                                                                     | 3000/s                          |
| Cost                 | Dependent of Cluster Size[[10]](#10)                                                                                                                                     | Dependent of volume[[9]](#9)                                                                                        | Very High                     |
| Partitioning         | Yes                                                                                                                                     | Yes                                                                                         | Yes. By group id[[3]](#3)     |
| Latency              | Milliseconds for some set-ups.<br /> Benchmarking showed ~2 ms median latency[[4]](#4) <br /> or 5 ms(200 MB/s load)[[5]](#5)           | 200 ms to 5 seconds                                                                         | ~100 ms[[11]](#11)                          |
| Replication          | Configurable replicas. Acknowledgement of message published can be on send,<br /> on receipt or on successful replication (local only). | Hidden (across three zones). Message published acknowledgement is always after replication. | No                          |

<small>Fonte: Diversos estudos. Agregado neste post [[6]](#10)</small>

### Messaging Guarantess
1. <i>**At-least-once delivery** — In the event a system fails or a network issue occurs, a message producer may continue to retry a message until it receives a successful acknowledgement. This can cause message duplication in the streaming platform. Hence, consumer applications must handle these scenarios by explicitly de-duplicating messages.</i><br />
2. <i>**At-most-once delivery** — If message producers are configured to not retry messages, it can lead to data loss if the streaming platform fails to commit and acknowledge the message. Consumers are only guaranteed messages that were successfully written to the streaming platform. Data loss is a serious problem for most businesses but its significance can vary based on your use case, specially if the original message can be recovered or reproduced easily.</i><br />
3. <i>**Exactly-once delivery** — The perfect world where a producer sends a message, it is written exactly once to the streaming platform and no duplication of messages or data loss occurs when a consumer reads that message.</i><br />

Nota: Apache Kafka, Kinesis e SQS FIFO trabalham com <i>At-least-once delivery</i> porém o Apache Kafka MSK pode trabalhar com <i>Exaclty-once delivery</i>[[8]](#8)

### Racional

Interpretamos:
1. No âmbito de performance (Latency, Throughput, Replication, etc) o Kafka demonstra uma superioridade pequena em relação ao Kinesis e muito grande ao SQS FIFO.
2. No âmbito técnico (Specialized Staff and Good practices) o SQS FIFO é absolutamente mais simples e o Kafka mais complexo. Porém o Kafka pode combater estas dificuldades utilizando o MSK.
3. No âmbito de custo. Kafka puro seria o mais "barato" (Considerando que a CIP ja possui staff com conhecimentos em Kafka) e o MSK é o segundo mais barato.
4. No âmbito de Conectividade o MSK possui a excelente qualidade do Kinesis e SQS em conectar com outras ferramentas da AWS.

Buscamos:
1.  altamente customizável e técnicamente viável;
2.  confiável (Replication e Partitioning);
3.  comunicação facil entre as ferramentas da AWS;
4.  custo acessível;
5.  casos de sucesso no mercado;
    
Decidimos: **Option 1: Apache Kafka MSK** <br /><br />

### Option 1: Apache Kafka MSK (Recommended)

"Apache Kafka is an open-source distributed event streaming platform used by thousands of companies for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications." - Apache Kafka Docs

### Option 2: AWS Kinesis

"O Amazon Kinesis facilita a coleta, o processamento e a análise de dados de streaming em tempo real, permitindo que você obtenha insights oportunos e reaja rapidamente às novas informações. O Amazon Kinesis oferece recursos essenciais para processar dados de streaming em qualquer escala de forma econômica, além da flexibilidade de escolher as ferramentas mais adequadas aos requisitos dos aplicativos. Com o Amazon Kinesis, você pode consumir dados em tempo real como vídeo, áudio, logs de aplicativos, clickstreams de sites e dados de telemetria de IoT para machine learning, análises e outros aplicativos. O Amazon Kinesis permite processar e analisar dados assim que são recebidos e responder instantaneamente, em vez de aguardar a conclusão da coleta de dados para poder iniciar o processamento."

### Options 3: SQS with SNS

"O Amazon Simple Queue Service (SQS) é um serviço de filas de mensagens gerenciado que permite o desacoplamento e a escalabilidade de microsserviços, sistemas distribuídos e aplicações sem servidor. O SQS elimina a complexidade e a sobrecarga associadas ao gerenciamento e à operação de middleware orientado a mensagens, além de permitir que os desenvolvedores se dediquem a criar diferenciais. Use o SQS para enviar, armazenar e receber mensagens entre componentes de software em qualquer volume, sem perder mensagens ou precisar que outros serviços estejam disponíveis. Comece a usar o SQS em minutos usando o Console AWS, a Interface da Linha de Comando ou o SDK preferido, juntamente com três comandos simples.

O SQS oferece dois tipos de filas. As filas padrão oferecem throughput máximo, o melhor esforço de classificação e entrega pelo menos uma vez. As filas FIFO do SQS são criadas para garantir que as mensagens serão processadas exatamente uma vez, na ordem exata em que forem enviadas."

## Consequences

**Pros**

- Reduz acoplamento entre serviços.
- Possibilita escalabilidade.
- Aumenta a resiliência na comunicação entre serviços.

**Cons**

- Aumenta a complexidade de desenvolvimento
- Exige a adoção de padrões de projeto fundamentais para garantir a consistência: Deduplication, Idempotency, etc.
- Adiciona mais uma Tech na Stack.

**Related Quality Attributes**

- <i>Reliability</i>
- <i>Performance Efficiency </i>

## Notes and Citations

<a id="1">[1]</a> <a href="https://docs.microsoft.com/pt-br/azure/architecture/patterns/pipes-and-filters">Pipes and filters:</a> Decompor uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados. Isso pode melhorar o desempenho, escalabilidade e reutilização, permitindo que os elementos de tarefa que executam o processamento sejam implantados e escalados de forma independente. <br />

<a id="2">[2]</a> <a href="https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines">Throughput study</a> <br />

<a id="3">[3]</a> <a href="https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/partitions-and-data-distribution.html">Partitions with SQS and SNS</a> <br />

<a id="4">[4]</a> <a href="https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines">Kafka latency</a> <br />

<a id="5">[5]</a> <a href="https://www.confluent.io/blog/kafka-fastest-messaging-system/#:~:text=For%20use%20cases%20that%20demand,with%20no%20overhead%20of%20replication.">Kafka vs. RabbitMQ</a> <br />

<a id ="6">[6]</a> <a href="https://blog.scottlogic.com/2018/04/17/comparing-big-data-messaging.html"> <i>Comparing Apache Kafka, Amazon Kinesis, Microsoft Event Hubs and Google Pub/Sub</i></a> traz uma elaborada comparação entre multiplos brokers e <a href="https://www.softkraft.co/aws-kinesis-vs-kafka-comparison/#:~:text=Performance%2Dwise%2C%20Kafka%20has%20a,still%20solidly%20in%20the%20thousands.">AWS Kinesis vs. Kafka</a> efetua uma comparação direta entre Kinesis e Kafka(MSK) e consolida, em alto nível, um racional de seleção de técnologia

<a id="7">[7]</a> <a href="https://docs.aws.amazon.com/msk/latest/developerguide/bestpractices.html">Kafka best practices</a>

<a id="8">[8]</a> <a href="https://www.confluent.io/blog/transactions-apache-kafka/">Kafka Transactions </a>

<a id="9">[9]</a> <a href="https://aws.amazon.com/pt/kinesis/data-streams/pricing/">Kinesis pricing</a>
<a id="10">[10]</a> <a href="https://aws.amazon.com/pt/msk/pricing/">Apache MSK pricing</a>
<a id="11">[11]</a> <a href="https://softwaremill.com/amazon-sqs-performance-latency/#:~:text=Single%2Dnode,message%20to%20travel%20through%20SQS).">Latency of SQS is 100ms to 150ms</a>