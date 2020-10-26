# Define role-based access control using Elasticsearch Security

Nesse objetivo você deve ser capaz de criar perfis de acesso (_roles_) e usuários que utilizem esses perfis de acesso.

A [documentação][security-apis] sobre como criar perfis de acesso e usuários no `elasticsearch` pode ser encontrada procurando por **security apis** na barra de pesquisa.

## criando perfil de acesso

No `elasticsearch` temos duas formas de criar perfis de acesso, através da API `/_security/role` e através do arquivo `roles.yml`.

A definição dos perfis de acesso utilizando o arquivo `roles.yml` é indicada apenas para alguns casos específicos, a forma recomendada é através da API `/_security/role`.

A criação de um perfil de acesso via API é feita através de um request `POST` para o endpoint da API, seguido do nome do perfil do payload com a definição do perfil.

```
POST /_security/role/adm-test
{
    payload
}
```

O _payload_ da definição do perfil de acesso tem a seguinte estrutura.

```
{
  "run_as": [ ... ], 
  "cluster": [ ... ], 
  "global": { ... }, 
  "indices": [ ... ], 
  "applications": [ ... ] 

}
``` 
- `run_as`: uma lista de usuários que podem ser personificados (_impersonate_) pelos usuários com esse perfil de acesso.
- `cluster`: privilégios do perfil de acesso no nível do cluster.
- `global`: privilégios do perfil de acesso no nível global.
- `indices`: privilégios do perfil de acesso no nível dos índices.
- `applications`: privilégios do perfil de acesso no nível das aplicações.

Apenas o item `run_as` é obrigatório, os outros podem estar presentes ou não de acordo com as necessidades de cada perfil de acesso.

A [documentação][define-roles] com uma explicação mais aprofundada de cada item pode ser encontrada pesquisando por **defining roles** na barra de pesquisa.

Como exemplo vamos criar um perfil de acesso chamado `adm-test` que terá acesso total aos índices `adm-teste-*` apenas e não poderá personificar nenhum usuário.

`REQUEST`
```
POST /_security/role/adm-test
{
  "run_as": [ ],
  "indices": [
    {
      "names": [ "adm-test-*" ],
      "privileges": [ "all" ]
    }
  ]
}
```

`RESPOSTA`
```
{
    "role": {
        "created": true
    }
}
```

Podemos verificar que o perfil foi criado consultando a API.

`REQUEST`
```
GET /_security/role/adm-test
```

`RESPOSTA`
```
{
    "adm-test": {
        "cluster": [],
        "indices": [
            {
                "names": [
                    "adm-test-*"
                ],
                "privileges": [
                    "all"
                ],
                "allow_restricted_indices": false
            }
        ],
        "applications": [],
        "run_as": [],
        "metadata": {},
        "transient_metadata": {
            "enabled": true
        }
    }
}
```

## criando usuário e atribuindo perfil de acesso

Agora que já criamos um perfil de acesso, vamos criar um usuário que irá utilizar esse perfil de acesso.

Vamos criar um usuário chamado `adm` com o perfil `adm-test`, para isso utilizamos o request a seguir.

`REQUEST`
```
POST /_security/user/adm
{
  "password" : "adm1234",
  "roles" : [ "adm-test" ],
  "full_name" : "Usuario Teste"
}
```

`RESPOSTA`
```
{
    "created": true
}
```

## validando permissões

Vamos validar as permissões tentando fazer um request com o usuário `adm` para um endpoint que ele não está autorizado.

`REQUEST`
```
$ curl --user adm:adm1234 localhost:9200/_cluster/health?pretty
```

`RESPOSTA`
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "action [cluster:monitor/health] is unauthorized for user [adm]"
      }
    ],
    "type" : "security_exception",
    "reason" : "action [cluster:monitor/health] is unauthorized for user [adm]"
  },
  "status" : 403
}
``` 
Vamos agora testar a criação de um índice seguindo o padrão _adm-test-*_

`REQUEST`
```
$ curl --user adm:adm1234 -X PUT "localhost:9200/adm-test-01/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
    "user" : "adm",
    "post_date" : "2020-10-25T10:00:00",
    "message" : "testando perfil de acesso"
}
'
```

`RESPOSTA`
```
{
  "_index" : "adm-test-01",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

Essa resposta confirma que o índice foi criado e o documento inserido, para validar vamos realizar uma pesquisa nesse índice.

`REQUEST`

```
$ curl --user adm:adm1234 es01:9200/adm-test-01/_search?pretty
```

`RESPOSTA`
```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "adm-test-01",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "user" : "adm",
          "post_date" : "2020-10-25T10:00:00",
          "message" : "testando perfil de acesso"
        }
      }
    ]
  }
}
```

Como última validação, vamos tentar criar um índice com um padrão diferente do definido no perfil de acesso.

`REQUEST`
```
$ curl --user adm:adm1234 -X PUT "localhost:9200/adm-prod-01/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
    "user" : "adm",
    "post_date" : "2020-10-25T10:00:00",
    "message" : "testando perfil de acesso"
}
'
```

`RESPOSTA`
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "action [indices:admin/create] is unauthorized for user [adm]"
      }
    ],
    "type" : "security_exception",
    "reason" : "action [indices:admin/create] is unauthorized for user [adm]"
  },
  "status" : 403
}
```

Por fim vamos deletar o índice criado anteriormente.

`REQUEST`
```
$ curl --user adm:adm1234 -X DELETE "localhost:9200/adm-test-01?pretty"
```

`RESPOSTA`
```
{
  "acknowledged" : true
}
```


[security-apis]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api.html
[define-roles]: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/defining-roles.html