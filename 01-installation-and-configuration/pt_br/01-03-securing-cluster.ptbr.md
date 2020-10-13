# Secure a cluster using Elasticsearch Security

Nesse objetivo é necessário saber como ativar a segurança em um cluster `elasticsearch`, tanto relacionada a autenticação de usuários quanto a comunicação entre os clusters.

A [documentação][secure-cluster] sobre como habilitar a segurança no `elasticsearch` pode ser encontrada buscando por **secure a cluster** na barra de pesquisa.

## configurações de segurança

No `elasticsearch` existem basicamente dois tipos de configurações de segurança, a que está relacionada com a autenticação de usuários e a que está relacionada com a comunicação entre os nós de um cluster.

No caso da autenticação de usuários, precisamos criar usuários e perfis de acesso (roles) especificando o que cada usuário pode fazer, como por exemplo, criar e deletar indices, ou apenas consultar.

No caso da comunicação entre os nós de um cluster, precisamos criar certificados para a comunicação TLS entre os nós.

Quando nosso cluster tem mais de um nó, é obrigatório utilizar certificados para a comunicação entre os nós.

Para configurarmos a segurança em um cluster `elasticsearch` iremos nos basear no tutorial disponível na [documentação oficial][tutorial]

## criando certificados

Nesse exemplo temos três nós de `elasticsearch` que formam um cluster chamado `cluster-es` e irão se comunicar usando TLS na camada de transporte.

|nome do nó|ip do nó|
|-----|-----|
|es01|10.0.1.31|
|es02|10.0.1.32|
|es03|10.0.1.33|

O primeiro passo é gerar o certificado para um dos nós, no caso iremos utilizar o nó **es01**.

```
# mkdir /etc/elasticsearch/certs
# cd /usr/share/elasticsearch
# ./bin/elasticsearch-certutil ca
```
Ao executarmos o comando `elasticsearch-certutil ca` seremos perguntados pelo nome do arquivo para a Autoridade Certificadora (CA) local que iremos usar e na sequência será pedido a criação de uma senha.

Iremos salvar o arquivo em `/etc/elasticsearch/certs/cluster-es-ca.p12`.

Na sequência iremos criar o certificado para o nó `es01`.

```
# ./bin/elasticsearch-certutil cert --ca /etc/elasticsearch/certs/cluster-es-ca.p12 --dns es01 --ip 10.0.1.31 --out /etc/elasticsearch/certs/es01.p12
```

Será solicitado a senha gerada no item anterior e uma nova senha para o certificado do nó.

Agora temos que adicionar as configurações de segurança no arquivo `elasticsearch.yml` e criar um `keystore` com a senha do certificado.

As configs necessárias são:

```
xpack.security.enabled: true                                                                          
xpack.security.transport.ssl.enabled: true                                                            
xpack.security.transport.ssl.keystore.path: /etc/elasticsearch/certs/es01.p12                         
xpack.security.transport.ssl.truststore.path: /etc/elasticsearch/certs/es01.p12
```

Para criar o keystore utilizamos os comandos abaixo.

```
# ./bin/elasticsearch-keystore create 
# ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
# ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```
Caso o `keystore` já existe, podemos usar só os comandos para adicionar as chaves.

Antes de iniciar o cluster precisamos criar os certificados para os outros dois nós, ainda a partir do nó **es01**.

```
# ./bin/elasticsearch-certutil cert --ca /etc/elasticsearch/certs/cluster-es-ca.p12 --dns es02 --ip 10.0.1.32 --out /etc/elasticsearch/certs/es02.p12
# ./bin/elasticsearch-certutil cert --ca /etc/elasticsearch/certs/cluster-es-ca.p12 --dns es03 --ip 10.0.1.33 --out /etc/elasticsearch/certs/es03.p12
```

Na sequência copiamos os certificados de cada nó paro servidor específico, adicionamos as configs no arquivo `elasticsearch.yml` e adicionamos as chaves no `keystore`.

Após o ajuste das configurações podemos subir o cluster e configurar os usuários.

## configurando os usuários

Para configurarmos os usuários internos, utilizamos o seguinte comando a partir de qualquer um dos nós ativos.

```
# cd /usr/share/elasticsearch
# ./bin/elasticsearch-setup-passwords interactive
```

Será solicitado a criação de uma senha para cada um dos usuários internos.

Após criarmos as senhas podemos verificar que o cluster está no ar com o comando a seguir.

```
# curl --user elastic:SENHA http://10.0.1.31:9200/_cluster/health?pretty
```

O resultado deve ser semelhante a mensagem a seguir.

```
{
  "cluster_name" : "cluster-es",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1,
  "active_shards" : 2,
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

## criptografia com clientes

Os passos anteriores configuram e ativam a comunicação criptografada apenas entre os nós de um cluster, é possível também configurar para que a comunicação entre o cluster e os clientes seja criptografada da mesma forma.

O procedimento é semelhante e é descrito nesse link da [documentação oficial][secure-client]


[secure-cluster]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/secure-cluster.html
[tutorial]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/encrypting-internode-communications.html
[secure-cliente]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/configuring-tls.html#tls-http