---
title: "Kafka SSL - Custom Principal Builder"
date: "2018-02-27"
categories: 
  - "apache-kafka"
tags: 
  - "apache-ranger"
  - "authentication"
  - "authorization"
  - "hdf"
  - "hdp"
  - "ssl"
---

I just pushed [a repository on Github](https://github.com/pvillard31/kafka-ssl-principal-builder) with code for a Custom Principal Builder when exposing Kafka brokers with SSL only.

The motivation behind this code is the following: some producers/consumers might not be able to use Kerberos to authenticate against Kafka brokers and, consequently, you can't use SASL\_PLAINTEXT or SASL\_SSL. Since PLAINTEXT is not an option (for obvious security reasons), it remains SSL.

When configuring a broker to use SSL, you will have authentication AND encryption if and only if 2-ways SSL is configured (by setting `ssl.client.auth=required`). It is **strongly** recommended to always set this property to **required**. (For additional information: [https://docs.confluent.io/current/kafka/authentication\_ssl.html#ssl-overview](https://docs.confluent.io/current/kafka/authentication_ssl.html#ssl-overview)).

When 2-ways SSL is enabled, the client will be authenticated using the Subject of the client certificate used to perform the handshake. It means that if the Subject is: `CN=kafkaClient, OU=OrgUnit, O=My Company`, you will have to set your topic ACLs to allow this user to consume/publish data.

When using [Apache Ranger](https://ranger.apache.org/) to manage authorizations for your Kafka brokers, it's not great to have this kind of username... That's why we want to define a custom principal builder to extract a username from the Subject of the certificate.

In the provided code, I want to expose two properties: a pattern that will be used to match the Subject and extract any capture group I want, and a value to construct the username using the capture groups. I added the two below properties in the `server.properties` configuration file on brokers side:

```
kafka.security.identity.mapping.pattern.dn=^.*[Cc][Nn]=([a-zA-Z0-9.]*).*$
kafka.security.identity.mapping.value.dn=$1
```

In this case, I only want to extract the CN part of the Subject and use it as the username of the client. If needed, I could use more complex patterns but with my previous example, my client would be authenticated with `kafkaClient` as username. It's now easier to define the authorizations on my topic using built-in ACLs or using Apache Ranger.

_Note:_ with Kafka 1.0+, the implementation changed a bit. Even though this code remains valid, there is a new interface that is much easier to implement ([https://cwiki.apache.org/confluence/display/KAFKA/KIP-189%3A+Improve+principal+builder+interface+and+add+support+for+SASL](https://cwiki.apache.org/confluence/display/KAFKA/KIP-189%3A+Improve+principal+builder+interface+and+add+support+for+SASL)) and it also provides the possibility to implement the principal builder when using SASL. For a Kafka 1.x version of this code, have a look on [this branch](https://github.com/pvillard31/kafka-ssl-principal-builder/tree/kafka_1.x).
