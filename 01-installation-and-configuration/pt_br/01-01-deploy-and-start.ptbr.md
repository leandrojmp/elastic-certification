## Deploy and start an Elasticsearch cluster that satisfies a given set of requirements

Esse objetivo indica que você deve saber como realizar a instalação e incialização de um cluster Elasticsearch que atenda um determinado conjunto de requisitos.

Um cluster Elasticsearch consiste de um ou mais nós `elasticsearch` com a mesma config de `cluster.name` e que consigam se comunicar entre si.

Os requisitos solicitados podem estar relacionados a configurações específicas do cluster e dos nós, por exemplo nome dos nós, nome do cluster, tipo de nós, nó master, quantidade de masters etc.

### Instalação

A [documentação][doc-install] referente a instalação pode ser encontrada buscando pela palavra **Install** na barra de pesquisa.

A forma mais simples de se instalar o elasticsearch é utilizando os gerenciadores de pacote dos sistemas.

- `yum` ou `dnf`, para sistemas baseados em Red Hat
- `apt`, para sistemas baseados em Debian
- `brew`, para o macOS, utilizando o Homebrew
- `msi`, para sistemas Windows (essa opção ainda está em versão beta)

Além disso é possível instalar o elasticsearch através de arquivos compactados.

- `.tar.gz`, para sistemas GNU/Linux e macOS
- `.zip`, para sistemas Windows

Existe também a opção de se utilizar o elasticsearch através de containers `docker`.

Devido as restrições de tempo e acesso para a certificação e ao que foi exibido nos webinars, acredito que não será necessário realizar a instalação do elasticsearch, apenas saber como iniciar manualmente a instância e suas principais configurações.

### Exemplo

Nesse exemplo vamos criar um cluster com dois nós, sendo que um deles será apenas master e o outro apenas nó de dados, para isso iremos instalar utilizando os arquivos `.tar.gz` em um sistema `CentOS` versão `7.8`, onde foi criado um usuário chamado **elastic** para esse processo.

O primeiro passo é baixar o arquivo `.tar.gz` da versão `7.2`. ([elasticsearch-7.2.1-linux-x86_64.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-linux-x86_64.tar.gz))

```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-linux-x86_64.tar.gz
```

Em seguida descompactamos o arquivo.

```
$ tar xvfz elasticsearch-7.2.1-linux-x86_64.tar.gz
```

Será criado um diretório chamado `elasticsearch-7.2.1`, para simplificar iremos renomear essa diretório como `elasticsearch`.

```
$ mv elasticsearch-7.2.1 elasticsearch
```

Agora vamos configurar o primeiro nó, chamado de `es01`, como apenas `master` e pertencente ao cluster chamado `cluster-es`, para isso editamos o arquivo `elasticsearch.yml` dentro do diretório `config`.

```
$ cd elasticsearch
$ vim config/elasticsearch.yml
```

`elasticsearcy.yml` do nó `es01`:
```
cluster.name: cluster-es
node.name: es01
discovery.seed_hosts: ["es01"]
cluster.initial_master_nodes: ["es01"]
network.host: _site_
node.master: true
node.data: false
node.ingest: false
node.ml: false
cluster.remote.connect: false
```
- `cluster.name`: define o nome do cluster. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster.name.html)
- `node.name`: define o nome do nó. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/node.name.html)
- `discovery.seed_hosts`: define quais os hosts que são elegíveis a master no cluster. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/discovery-settings.html)
- `cluster.initial_master_nodes`: define quais os hosts elegíveis a master participam da eleição inicial do cluster. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/discovery-settings.html)
- `network.host`: indica qual o ip a instância irá utilizar. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-network.html)
- `node.master`: indica se o nó pode ser do tipo master. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
- `node.data`: indica se o nó pode ser do tipo data. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
- `node.ingest`: indica se o nó pode ser do tipo ingest. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
- `node.ml`: indica se o nó pode ser do tipo ml (machine learning). [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
- `cluster.remote.connect`: habilita ou desabilita a busca cross-cluster. [doc](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)

Repetindo o processo para o segundo nó, chamado de `es02`, configurado como apenas do tipo `data` e pertencente ao cluster chamado `cluster-es`, teremos o seguinte arquivo de config.

`elasticsearcy.yml` do nó `es02`:
```
cluster.name: cluster-es
node.name: es02
discovery.seed_hosts: ["es01"]
cluster.initial_master_nodes: ["es01"]
network.host: _site_
node.master: false
node.data: true
node.ingest: false
node.ml: false
cluster.remote.connect: false
```
Como utilizamos a opção `network.host` nos arquivos de configuração, o `elasticsearch` entende que estamos trabalhando com um cluster de produção e para isso efetua uma série de verificações (*bootstrap checks*) antes de permitir a inicialização.

Essas verificações são descritas nesse [link da documentação][bootstrap] e estão relacionadas basicamente a [configurações importantes de sistema][system-config] que precisam ser feitas.

Se tentarmos iniciar as instâncias sem realizar essas alterações, iremos ver as seguintes linhas no log.

```
[2020-09-07T17:24:10,583][INFO ][o.e.b.BootstrapChecks    ] [es01] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

Os erros de bootstrap estão relacionados o número de *file descriptors* e a *memória virtual*.

Para solucionar o problema de *file descriptors*, adicionamos a linha abaixo no arquivo `/etc/security/limits.conf`, isso irá permitir que o usuário `elastic` utilize até `65535` *file descriptors*.

```
elastic  -  nofile  65535
```

Para solucionar o problema da *memória virtual* editamos o arquivo `/etc/sysctl.conf` e adicionamos a linha abaixo.

```
vm.max_map_count=262144
```

Lembrando que esses arquivos só podem ser alterados pelo `root` ou algum usuário com permissão para `sudo`.

Após essas alterações podemos inicializar o `elasticsearch` em ambos os nós e aguardar eles se comunicarem.

```
$ cd elasticsearch
$ bin/elasticsearch
```
Utilizando a API de [Cluster Health][cluster-health] podemos verificar o status do cluster.

```
$ curl http://es01:9200/_cluster/health?pretty
```

```
{
  "cluster_name" : "cluster-es",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```
Pela repostas podemos ver que o cluster tem 2 nós, sendo que somente 1 dele é do tipo `data`.

[doc-install]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/install-elasticsearch.html
[doc-nodes]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html
[bootstrap]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/bootstrap-checks.html
[system-config]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/system-config.html
[cluster-health]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-health.html