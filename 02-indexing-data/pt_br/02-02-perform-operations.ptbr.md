# Perform index, create, read, update, and delete operations on the documents of an index

Nesse objetivo é necessário saber criar, ler, atualizar e deletar documentos em um índice no `elasticsearch`.

A [documentação][docs-api] sobre os endpoints relacionados aos índices pode ser encontrada buscando por **documents api** na barra de pesquisa.

## índice de exemplo

No `elasticsearch` todo documento pertence a um índice, nesse exemplo iremos utilizar um índice chamado `example` definido da seguinte maneira.

```
PUT example
{
    "settings" : {
        "number_of_shards" : 1,
        "number_of_replicas": 2
    },
    "mappings" : {
        "properties" : {
            "name" : { "type" : "keyword" },
            "address" : { "type" : "text" }
        }
    }
}
```

A requisição acima define um índice chamado `example`, com **1** _shard_  primária e **2** réplicas, além disso definimos também um mapeamento para dois campos, `name` e `address`, sendo que o campo `name` é do tipo `keyword` e o campo `address` é do tipo `text`.

## criando, atualizando e deletando um documento

Para criarmos um documento no índice `example` utilizamos a seguinte requisição.

```
PUT example/_doc/1
{
    "name" : "leandro",
    "address" : "example street, 01"
}
```

Para atualizarmos o documento basta alterar um dos campos, ou adicionar um novo campo, e fazer uma requisição mantendo o _id_ do documento.

```
PUT example/_doc/1
{
    "name" : "leandro",
    "address" : "example street, 01, apto 200"
}
```

Se quisermos que uma nova requisição para um _id_ já existente não atualize o documento, devemos utilizar o endpoint `_create` na requisição.

```
PUT example/_create/1
{
    "name" : "leandro",
    "address" : "example street, 01, apto 200"
}
```

Dessa forma, se o documento com _id_ 1 não existir, ele será criado, caso ele exista, será retornado um erro informando que há um conflito de versão pois o documento já existe.

Para deletarmos um documento basta fazer uma requisição do tipo `DELETE` para o índice, passando o _id_ do documento.

```
DELETE example/_doc/1
```

A requisição acima marca o documento com _id_ 1 como deletado no índice.

## utilizando ids auto-gerados

Nos exemplos anteriores passamos o _id_ do documento na hora da criação do mesmo, mas podemos também deixar que o `elasticsearch` crie um _id_  aleatório, para isso precisamos fazer uma requisição do tipo `POST` na hora da criação.

```
POST example/_doc
{
    "name" : "leandro",
    "address" : "example street, 01"
}
```

Na resposta da criação teremos o _id_ gerado pelo `elasticsearch`.

```
{
    "_index": "example",
    "_type": "_doc",
    "_id": "Ose0GHcBGetHIyfUHaum",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

Se quisermos atualizar esse documento precisamos passar o _id_  auto-gerado na requisição.

```
POST example/_doc/Ose0GHcBGetHIyfUHaum
{
    "name" : "leandro",
    "address" : "example street, 01, apto 300"
}
```

Para deletar o documento basta passar o _id_ auto-gerado na requisição.

```
DELETE example/_doc/Ose0GHcBGetHIyfUHaum
```

## lendo um documento

Para lermos um documento podemos fazer uma requisição do tipo `GET` passando o _id_ do documento.

```
GET example/_doc/1
```

Como resposta temos um `JSON` com o documento no campo `_source` e alguns campos de metadados do `elasticsearch`.

```
{
    "_index": "example",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,
    "_seq_no": 6,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "leandro",
        "address": "example street, 01"
    }
}
```

Se quisermos apenas o conteúdo do documento, sem os campos de metadados, podemos fazer uma requisição solicitando apenas o `_source`.

```
GET example/_source/1
```

Como resposta teremos apenas os campos do documento.

```
{
    "name": "leandro",
    "address": "example street, 01"
}
```

Podemos consultar todas as opções desse endpoint na documentação sobre a [api GET][get-api]


[docs-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs.html
[get-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-get.html