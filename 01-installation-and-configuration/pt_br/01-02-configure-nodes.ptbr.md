# Configure the nodes of a cluster to satisfy a given set of requirements

Esse objetivo indica que você deve saber quais os tipos de nós possíveis para cada instância `elasticsearch` em um cluster, a diferença entre eles e como configurar cada instância para se comportar como determinado tipo de nó.

A [documentação][node-types] sobre os tipos de nós pode ser encontrada buscando pela palavra **node** na barra de pesquisa.

Existem 5 tipos diferentes de nós.

- **master**
- **data**
- **ingest**
- **machine learning**
- **coordinating**

## tipos de nós

### master

Um nó `master` é um nó responsável por criar e deletar índices, verificar quais nós fazem parte do cluster e decidir quais *shards* serão alocadas em quais nós.

A configuração `node.master: true` no arquivo `elasticsearch.yml` da instância `elasticsearch` torna esse nó elegível a master, *master elegible* , e ele participa da votação com os outros nós para definar qual será o no master do cluster, caso esse nó caia, os outros nós elegíveis a master realizam uma nova votação para definir qual será o novo master.

### data

Um nó do tipo `data` é um nó que armazena os dados dos índices do cluster e realiza operações relacionadas aos dados como operações *CRUD*, pesquisas e agregaçoes, essas operações utilizam bastante os recursos de CPU, I/O e Memória, por isso é importante monitorar o desempenho dos nós do tipo `data` e adicionar novos nós caso se verifique sobrecargas.

Para configurarmos um nó como sendo do tipo `data` usamos `node.data: true`.

### ingest

Um nó do tipo `ingest` é um nó que pode realizar um pré-processamento nos dados antes da indexação, como por exemplo analisar a mensagem e criar novos campos, uma função similar ao logstash.

A [documentação][ingest-node] mais completa sobre os nós do tipo `ingest` pode ser encontrada buscando por **ingest node** na barra de pesquisa.

Para configurarmos um nó como sendo do tipo `ingest` usamos `node.ingest: true`.

### machine learning

Um nó do tipo `machine learning` ou `ml`, é um nó que pode rodar jobs de *machine learning* e responder aos requests da API *machine learning*.

Para configurarmos um nó como sendo do tipo `machine learning` usamos `node.ml: true`.

Esse tipo de nó precisa de uma licença do tipo **platinum** ou **enterprise** portanto para usarmos suas funcionalidades precisamos também da opção `xpack.ml.enabled: true`.

### coordinating

Um nó do tipo `coordinating` é um nó que não realiza nenhuma das outras funcões e é responsável apenas por receber e direcionar os requests e gerenciar a fase de pesquisa.

Ter nós apenas de coordenação ajuda a alivar os clusters `master` e de `data` em alguns casos.

## configurações.

Por padrão toda instância realiza todas as funções `master`, `data`, `ingest`, `ml` e `coordinating`.

Em alguns casos dependendo do tamanho do cluster é interessante ter nós especializados em algumas das funções possíveis.

### master only

Para termos um nó que seja apenas `master`, usamos a seguinte configuração.

```
node.master: true
node.data: false
node.ingest: false
node.ml: false
cluster.remote.connect: false
```

### data only

Para termos um nó que seja apenas do tipo `data` usamos a seguinte configuração.

```
node.master: false
node.data: true
node.ingest: false
node.ml: false
cluster.remote.connect: false
```

### ingest only

Para termos um nó que seja apenas do tipo `ingest` usamos a seguinte configuração.

```
node.master: false
node.data: false
node.ingest: true
node.ml: false
cluster.remote.connect: false
```

### ml only

Para termos um nó que seja apenas do tipo `ml` usamos a seguinte configuração.

```
node.master: false
node.data: false
node.ingest: false
node.ml: true
xpack.ml.enabled: true
cluster.remote.connect: false
```

### coordinating only

Para termos um nó que seja apenas de coordenação usamos a seguinte configuração.

```
node.master: false
node.data: false
node.ingest: false
node.ml: false
cluster.remote.connect: false
```

[node-types]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html
[ingest-node]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html