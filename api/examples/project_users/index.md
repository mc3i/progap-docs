[API](..) / Utilisateurs / Liste des affaires

# API : Utilisateurs / Liste des affaires

Cette documentation présente les requêtes GraphQL pour récupérer la liste des affaires (projets et devis) accessibles par utilisateur selon leurs permissions.

## Contexte

L'API GraphQL PROGAP utilise le système de privilèges basé sur les vues `mdb_privilege_project_user` et `mdb_privilege_estimate_user` qui gèrent automatiquement :

- Les permissions directes assignées aux utilisateurs
- L'héritage des permissions via les espaces de travail (workspaces)
- Les rôles associés pour chaque affaire/devis

## Requête combinée : Projets et devis pour un utilisateur

Pour récupérer à la fois les projets et les devis accessibles à un utilisateur en une seule requête :

### Requête GraphQL

Query

```graphql
query GetUserBusinessItems($user_ids: [uuid!]!) {
  affaires: mdb_privilege_project_user(where: { user_id: { _in: $user_ids } }) {
    user_id
    project_id
    role_id
    user {
      firstname
      lastname
      mail
    }
    project {
      code
      name
      workspace {
        name
      }
    }
  }
  devis: mdb_privilege_estimate_user(where: { user_id: { _in: $user_ids } }) {
    user_id
    estimate_id
    role_id
    user {
      firstname
      lastname
      mail
    }
    estimate {
      code
      name
      project {
        code
        name
      }
    }
  }
}
```

Query variables

```json
{
  "user_ids": ["ce2445a1-7262-444d-b368-201e54dad87c"]
}
```
