# Define an index that satisfies a given set of requirements

Nesse objetivo é necessário saber o que é um índice no `elasticsearch` e quais suas características.

A [documentação][index-api] sobre os endpoints relacionados aos índices pode ser encontrada buscando por **indices api** na barra de pesquisa.

## índices no elasticsearch

No `elasticsearch` todos os dados ficam armazenados em um ou mais _indices_ e esses _indices_ podem ter diferentes caractéristicas entre si, sendo que as principais características de um índice seriam:

- número de _shards_, sendo divididas entre primárias e réplicas
- mapeamento

## shards e replicas

Um índice no `elasticsearch` é na verdade um agrupamento lógico de uma ou mais _shards_ físicas armazenadas no disco, cada _shard_ é um índice auto-contido, isso permite que os documentos em um índice sejam distribuídos entre diversas _shards_ e essas _shards_ podem ser distribuídas entre diversos nodes.

As _shards_  podem ser primárias ou réplicas, a principal diferença entre elas é que somente as _shards_ primárias realizam a indexação dos documentos, que são então copiados para as réplicas e tanto as réplicas quanto as primárias podem responder a requisições de pesquisa, além disso a definição do número de _shards_ primárias é feita durante a criação do índice e não pode ser alterada sem que o índice seja recriado, já o número de réplicas pode ser alterado dinamicamente de acordo com as necessidades.

Um outro ponto importante é que as réplicas sempre serão armazenadas em um nó diferente de onde está armazenada a primária.

## mapeamento

O mapeamento de um índice consiste em especificar qual o tipo de dado de cada campo para que as pesquisas sejam feitas da melhor forma possível.

Quando não definimos um mapeamento para um índice, o `elasticsearch` infere o tipo de dado do campo de acordo com o primeiro valor que ele recebe para aquele campo em um documento, para campos com valores alfanuméricos, ele acaba definindo o campo como sendo tanto do tipo `keyword` quanto do tipo `text` o que pode levar a um uso desnecessário de espaço, além disso alguns formatos de data fora do padrão podem não ser identificados como sendo do tipo `date`, sendo indexados como `keyword` e `text` ou algum tipo numérico.

## definindo um índice

Como exemplo, vamos definir um índice chamado `example`, com 2 _shards_  primárias, 1 réplica para cada _shard_, com um campo chamado `name` do tipo `keyword` e um campo chamado `address` do tipo `text`.

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

Com esse request criamos o índice com essas definições, mas ainda sem nenhum documento.

[index-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices.html