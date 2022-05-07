---
title: "Scale from Zero to Millions of Users"
tags:
- system-design
---


## Single Server Setup
The Design of every complex system starts with something simple. Consider a single server setup where everything is running in on one server: web, app, DB, cache, etc. as shown below.
![[Excalidraw/Drawing 2022-04-30 21.28.47.excalidraw.png]]

## Basic Flow
To understand this setup, we can inspect **request flow** and **traffic source**.
![[Excalidraw/Drawing 2022-04-30 21.57.17.excalidraw.png]]

### Request flow
1. Users access the server using domain names from the browser/app
2. [[DNS]] resolves the domain name to a valid [[IP]] address
3. An [[IP]] address is returned to the browser/app
4. Browser/app uses the [[IP]] address to send [[HTTP]] requests to your server
5. The Web server returns [[HTML]] pages or [[JSON]] response

### Traffic source
1. **Web application:** It uses a combination of server-side languages (Java, Nodejs, etc) to handle business logic, storage, etc, and client-side languages (HTML and Javascript) for presentation.
2. **Mobile application:** A client-side app(Android, iOS, etc) running on a mobile device uses [[HTTP]] protocol to communicate with the webserver. [[JSON]] is a commonly used response format because of if simplicity.

## Database
Growing user base requires more servers. The most basic segregation we can do here is separating web/mobile(web tier) and database(data tier) servers. This decoupling will allow them to be scaled independently.
![[Excalidraw/Drawing 2022-04-30 22.25.25.excalidraw.png]]

### Which databases to use?
We have choice between relational and non-relational databases.

Relational databases (RDBMS) or SQL databases represent and store data in tables and rows. You can perform join operation using SQL across different database tables.

Non-Relational databases or NoSQL databases usually don't support join operations & can be grouped into 4 categories:
- key-value stores
- graph stores
- column stores
- document stores

For most cases relational databases are best option given historical track record of over 40 years. There are some scenarios where non-relational databases might be the right choice. Those specific use cases are as follows:
- Your application requires super-low latency.
- Your data is unstructured, no relational data
- You only need to serialize and deserialize data (JSON, XML, YAML, etc.)
- You need to store a massive amount of data.

## Scaling : Vertical vs Horizontal
## Load balancer
## Database Replication
## Cache
### Cache tier
## Content Delivery Network (CDN)
## Stateful Architecture
## Stateless Architecture
## Data Centers
## Message Queue
## Logging, Metrics, Automation
## Database scaling
### Vertical scaling
### Horizontal scaling
## Conclusion: Millions of users and beyond

## References
- [[books/System Design Interview by Alex Xu]]

