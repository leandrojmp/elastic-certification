# elastic-certification

## pt-br
Repositório para armazenar notas e dicas durante o estudo auto-guiado para o [Elastic Certified Engineer Exam](https://www.elastic.co/training/elastic-certified-engineer-exam)

Como não existe nenhum exemplo de prova disponível, vou me guiar pelos objetivos presentes na [descrição da certificação](https://www.elastic.co/training/elastic-certified-engineer-exam).

# objetivos

## Installation and Configuration

- Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
- Configure the nodes of a cluster to satisfy a given set of requirements
- Secure a cluster using Elasticsearch Security
- Define role-based access control using Elasticsearch Security

## Indexing Data

- Define an index that satisfies a given set of requirements
- Perform index, create, read, update, and delete operations on the documents of an index
- Define and use index aliases
- Define and use an index template for a given pattern that satisfies a given set of requirements
- Define and use a dynamic template that satisfies a given set of requirements
- Use the Reindex API and Update By Query API to reindex and/or update documents
- Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

## Queries

- Write and execute a search query for terms and/or phrases in one or more fields of an index
- Write and execute a search query that is a Boolean combination of multiple queries and filters
- Highlight the search terms in the response of a query
- Sort the results of a query by a given set of requirements
- Implement pagination of the results of a search query
- Use the scroll API to retrieve large numbers of results
- Apply fuzzy matching to a query
- Define and use a search template
- Write and execute a query that searches across multiple clusters

## Aggregations

- Write and execute metric and bucket aggregations
- Write and execute aggregations that contain sub-aggregations
- Write and execute pipeline aggregations

## Mappings and Text Analysis

- Define a mapping that satisfies a given set of requirements
- Define and use a custom analyzer that satisfies a given set of requirements
- Define and use multi-fields with different data types and/or analyzers
- Configure an index so that it properly maintains the relationships of nested arrays of objects
- Configure an index that implements a parent/child relationship

## Cluster Administration

- Allocate the shards of an index to specific nodes based on a given set of requirements
- Configure shard allocation awareness and forced awareness for an index
- Diagnose shard issues and repair a cluster's health
- Backup and restore a cluster and/or specific indices
- Configure a cluster for use with a hot/warm architecture
- Configure a cluster for cross cluster search

# webinars
Links para Webinars da Elastic com informações e questões simuladas.

## webinar 2019-01 (versão 6.5)
<iframe width="560" height="315" src="https://www.youtube.com/embed/dzo_uR3IsbQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## webinar 2019-04 (versão 7.2)
<iframe width="560" height="315" src="https://www.youtube.com/embed/hsaLZSKCkF0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>