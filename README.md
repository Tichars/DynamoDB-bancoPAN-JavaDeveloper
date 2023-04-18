# Banco PAN Java Developer


### Serviço utilizado
  - Amazon DynamoDB
  - Amazon CLI para execução em linha de comando

### Comandos para execução do experimento:


- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Movies \
    --attribute-definitions \
        AttributeName=Genre,AttributeType=S \
        AttributeName=Title,AttributeType=S \
    --key-schema \
        AttributeName=Director,KeyType=HASH \
        AttributeName=Title,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um filme

```
aws dynamodb put-item \
    --table-name Movies \
    --item file://itemmovie.json \
```

- Inserir em lotes

```
aws dynamodb batch-write-item \
    --request-items file://batchmovie.json
```

- Criar um index global secundário baseado no diretor do filme

```
aws dynamodb update-table \
    --table-name Movies \
    --attribute-definitions AttributeName=Director,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Director-index\",\"KeySchema\":[{\"AttributeName\":\"Director\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no nome do artista e no título do álbum

```
aws dynamodb update-table \
    --table-name Movies \
    --attribute-definitions\
        AttributeName=Genre,AttributeType=S \
        AttributeName=Director,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"GenreDirector-index\",\"KeySchema\":[{\"AttributeName\":\"Genre\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Director\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no título do filme e no ano de lançamento

```
aws dynamodb update-table \
    --table-name Movies \
    --attribute-definitions\
        AttributeName=Title,AttributeType=S \
        AttributeName=ReleaseYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ReleaseYear-index\",\"KeySchema\":[{\"AttributeName\":\"Title\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"ReleaseYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar filme por gênero

```
aws dynamodb query \
    --table-name Movies \
    --key-condition-expression "Genre = :genre" \
    --expression-attribute-values  '{":genre":{"S":"Drama"}}'
```
- Pesquisar filme por gênero e título

```
aws dynamodb query \
    --table-name Movies \
    --key-condition-expression "Genre = :genre and Title = :title" \
    --expression-attribute-values file://keyconditions.json
```

- Pesquisa pelo index secundário baseado no Diretor do filme

```
aws dynamodb query \
    --table-name Movies \
    --index-name Director-index \
    --key-condition-expression "Director = :name" \
    --expression-attribute-values  '{":name":{"S":"Francis Ford Copola"}}'
```

- Pesquisa pelo index secundário baseado no nome do diretor e genero do filme

```
aws dynamodb query \
    --table-name Movies \
    --index-name GenreDirector-index \
    --key-condition-expression "Genre = :v_genre and Director = :v_title" \
    --expression-attribute-values  '{":v_genre":{"S":""},":v_title":{"S":"The Dark Knight"} }'
```

- Pesquisa pelo index secundário baseado no título do filme e no ano de lançamento

```
aws dynamodb query \
    --table-name Movies \
    --index-name Title-index \
    --key-condition-expression "Title = :v_movie and ReleaseYear = :v_year" \
    --expression-attribute-values  '{":v_movie":{"S":"The Shawshank Redemption"},":v_year":{"S":"1994"} }'
```
