# [API](..) / Connexion à l'API via Postman
## Connexion
Depuis l'application Postman, créer une nouvelle requête.

- Renseigner "POST dans la méthode 
- Renseigner le nom de votre API dans l'URL (ex: `https://VOTRE_INSTANCE.mc3i.io/api/v1/graphql`)
- Renseigner l'entête `x-hasura-admin-secret` en utilisant la valeur transmise par M&C 3i

![Hasura GraphiQL page](../../resources/images/postman_headers.png)

## Effectuer une requête
Pour envoyer une requête, depuis l’onglet `Body`, sélectionner l’option GraphQL, renseigner une requête et les éventuelles variabes de la requête puis cliquer sur `Send`.

### Exemple de requête

```graphql
query MyQuery {
    mdb_progap_project {
        code
        name
    }
}
```


### Exemple de résutat

```json
{
    "data": {
        "mdb_progap_project": [
            {
                "code": "2023-A-BC",
                "name": "Piscine Longjumeau"
            }
        ]
    }
}
```