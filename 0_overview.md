# Overview

Search Guard® SSL is a plugin for Elasticsearch which provides SSL/TLS encryption and authentication for Elasticsearch. It offers:

* Node-to-node encryption through SSL/TLS (Transport layer)
* Secure REST layer through HTTPS (SSL/TLS)
* Supports JDK SSL and Open SSL
* Works with Kibana 4, logstash and beats

The only external dependency is Netty 4 (and Tomcat Native if Open SSL is used)

Search Guard SSL is the foundation layer of Search Guard, which adds authentication and authorization and is also available as an Elasticsearch plugin from floragunn.

If you just need to encrpyt your traffic, and make sure only trusted nodes and clients can join and access a cluster, Search Guard SSL is all you require. If you want authentication and authorization, you need both Search Guard SSL + Search Guard.

In this document the following abbreviations are being used:

* ES: Elasticsearch
* SG SSL: Search Guard SSL
* SG: Search Guard
 
This documentation refers only to Search Guard 2.0 and above. Search Guard for ES 1.x is not actively maintained anymore.

## For the impatient

If you just want to get up and running quickly, please refer to the [quickstart tutorial](2_quick_start.md)

## Motivation

While Elasticsearch is often used for storing and searching sensitive data, it does not offer encryption or authentication/authorization out of the box. While it is possible to implement an authentication/authorization plugin without SSL/TLS encryption, we strongly believe that this will only get you fake security. 

Obviously, by using SSL the traffic between ES nodes and the HTTP(S) traffic will be encrypted. Which means that

* you can be sure that nobody is spying on the traffic
* you can be sure that nobody tampered with the traffic

However, and this is equally important, by using SSL certificates you can also make sure that **only identified and trusted nodes or clients** can interact with the cluster.

For example, a malicious attacker might set up a new data node, join the cluster, effectively getting a copy of (at least) a portion of your data. Adding SSL, we can make sure that this attack scenario is not possible.

## SSL in a nutshell

*Note: While we explain how SG SSL uses SSL on the transport layer between nodes, these principles are also true for the ES REST layer.*

SSL is a cryptographic protocol to secure the communication over a computer network. It uses **symmetric encryption** to encrypt the transmitted data, by negotiating a unique secret at the start of any **SSL session** between the two participating parties. It uses **public key cryptography** 	to authenticate the identity of the participating parties.

SSL supports many different ways for exchanging keys and encrypting data, but that is beyond the scope of this document. We'll focus only on the main concepts necessary to set up SG SSL.

### Certificates

SSL uses certificates (called "SSL certificates" or "digital certificates") for encrypting traffic and for identifying and authenticating the two participating parties in an SSL session. 

### SSL handshake

Before any user data is sent between these parties, first an **SSL session** has to be established. This process is called **SSL handshake**.   

### Keystore and Truststore

SG SSL uses the concept of having a mandatory **keystore** and a **truststore** on each node.

The **keystore** holds the nodes private key and the associated certificate or certificate chain. The certificate is used to authenticate the node against other nodes. The private key is used during the SSL handshake for negotiating the master secret.

The **truststore** contains all trusted certificates, typically Root CAs and intermeditate certificates. 
