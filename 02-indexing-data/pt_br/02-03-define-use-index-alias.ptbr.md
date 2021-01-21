# Define and use index alias

`OBJETIVOS`

- Definir um alias de índice
- Utilizar um alias de índice


Um alias de index (_index alias_) é uma forma de se referir a um ou mais índices com um nome diferente, um apelido.

Podemos por exemplo ter um índice chamado `logs-server-0001` e criar um alias chamado `logs` para nós referir a esse índice.

Um alias não pode ter o mesmo nome de índice já existente

`PESQUISA`

palavras chave para encontrar [documentação][index-alias].
> index alias

## definindo um alias de índice

`REQUEST`

```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "indice-destino", "alias" : "nome-do-alias" } }
    ]
}
```

_OU_

```
PUT /indice-destino/_alias/nome-do-alias
```

## criando um alias para o índice example

`REQUEST`

```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "example", "alias" : "ex" } }
    ]
}
```

_OU_

```
PUT /example/_alias/ex
```

`RESPOSTA`

```
{
    "acknowledged": true
}
```

## utilizando um alias de índice

Para utilizarmos o alias de índice basta trocar o nome do índice no request pelo alias.

- `índice`: _example_
- `alias`: _ex_

`REQUEST`

```
GET /ex/_source/1
```

`RESPOSTA`

```
{
    "name": "leandro",
    "address": "example street, 01"
}
```

[index-alias]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html