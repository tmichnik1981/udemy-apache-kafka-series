# Confluent schema registry

#### Introduction

##### The need of schemas

- Kafka takes bytes as an input and publishes them
- No data verification
- We need data to be self describable
- We need data to be able to evolve data without breaking downstream consumers
- We need schemas and a schema registry
- The Schema Registry has to be a separate components
- Producers and Consumers need to be able to talk to it
- The Schema Registry must be able to reject bad data
- A common data format must be agreed upon
  - It needs to support schemas
  - It need to support evolution
  - It needs to be lightweight