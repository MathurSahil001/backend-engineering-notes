# Backend Engineering Notes

My long-term handbook for understanding backend systems, revising concepts, discussing engineering decisions in interviews, and recording lessons from building **FlashCart**, a high-scale e-commerce backend.

The notes start simple, then explain what happens internally, how a component fits into a running system, and what can fail. Examples favor Java, Spring Boot, PostgreSQL, Redis, Kafka, Docker, and microservices. This repository grows as I learn and debug real problems; it is not meant to be a finished textbook.

## Organization

| Folder | What belongs here |
| --- | --- |
| [`01-java/`](01-java/) | Java language, concurrency, JVM, and collections |
| [`02-spring/`](02-spring/) | Spring Boot, dependency injection, configuration, and web layers |
| [`03-web-http/`](03-web-http/) | HTTP, REST, API design, and network behavior |
| [`04-databases/`](04-databases/) | PostgreSQL, SQL, transactions, indexing, and database operations |
| [`05-jpa-hibernate/`](05-jpa-hibernate/) | Persistence mapping, sessions, fetching, and ORM behavior |
| [`06-docker/`](06-docker/) | Images, containers, networking, volumes, and local development |
| [`07-redis/`](07-redis/) | Caching, expiry, and Redis data structures |
| [`08-kafka/`](08-kafka/) | Events, producers, consumers, delivery, and operations |
| [`09-microservices/`](09-microservices/) | Service boundaries, communication, reliability, and deployment |
| [`10-system-design/`](10-system-design/) | Scale, architecture, tradeoffs, and end-to-end designs |
| [`11-debugging-journal/`](11-debugging-journal/) | Real incidents: symptoms, evidence, root cause, fix, and prevention |
| [`12-revision-sheets/`](12-revision-sheets/) | Short summaries for quick recall before practice or interviews |
| [`13-interview-questions/`](13-interview-questions/) | Practical prompts and discussion practice |

Start with [Docker and local PostgreSQL](06-docker/docker-foundations-and-postgresql-ports.md). Its compact companion is the [Docker revision sheet](12-revision-sheets/docker-and-postgresql.md), and the [port conflict case](11-debugging-journal/postgresql-port-conflict.md) records the debugging method.

## How to use this handbook

1. Read one topic and explain its mental model aloud without looking.
2. Try the example in FlashCart. Record what actually happened, including errors and fixes.
3. Answer the active recall questions. Use the revision sheet later for a fast refresher.
4. Update the existing topic when new evidence or understanding changes it. Create a new file only for a distinct concept.

## Note format

Topic notes follow these eleven sections: Core Idea, Mental Model, How It Fits In The Backend, Important Concepts, Engineering Decisions, Real Example, Common Mistakes, Debugging Lessons, Interview Discussion, 5-Minute Revision, and Active Recall Questions. The journal records observed facts separately from hypotheses. Commands are examples: adjust credentials, names, and ports to the actual environment. Never commit secrets.
