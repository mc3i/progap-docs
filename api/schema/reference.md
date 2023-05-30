[API](..) / Schéma / Référence

## Sommaire
- Gestion des licences (*en cours d'écriture*)
  - [Utilisateurs](#utilisateurs)
  - [Modèles de licences](#modèles-de-licence)
  - [Licences](#licenses)
  - [Synchronisation](#synchronisation)
- Gestion des utilisateurs (*en cours d'écriture*)
  - [Utilisateurs](#utilisateurs-1)
  - [Rôles](#rôles)
  - [Affectation](#affectation)
- Gestion des affaires (*en cours d'écriture*)
  - [Espaces de travail](#espaces-de-travail)
  - [Droits d'accès aux espaces de travail](#droits-daccès-aux-espaces-de-travail)
  - [Affaires](#affaires)
  - [Droits d'accès aux affaires](#droits-daccès-aux-affaires)

## Gestion des licences
### Utilisateurs
Le type `mdb_helios_user` correspond à un utilisateur sous licence PROGAP.

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_helios_user             | Liste des utilisateurs          |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| mail          | text, not null, unique        | Adresse e-mail              |
| firstname     | text, not null                | Prénom                      |
| lastname      | text, not null                | Nom                         |

### Modèles de licence
Le type `mdb_helios_license_template` correspond à un modèle de licence PROGAP.

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_helios_license_template | Liste des modèles de licence    |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| code          | text, not null, unique        | Code                        |
| name          | text, not null                | Désignation                 |

### Licenses
Le type `mdb_helios_license` correspond à une licence PROGAP.

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_helios_license          | Liste des licences              |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| user_id       | uuid, not null                | Identifiant de l'utilisateur `mdb_helios_user.id` |
| helios_role   | text, not null                | HELIOS_INSTANCE_MANAGER : gestionnaire de license<br>HELIOS_INSTANCE_USER : utilisateur |

### Synchronisation
Le type `mdb_helios_sync_user_license` permet de synchroniser les utilisateurs et les licences associées.
Ce type a été créé spécifiquement pour aider les services informatiques à déclarer automatiquement des licences.

| Type                         | Description                     |
|------------------------------|---------------------------------|
| mdb_helios_sync_user_license | Synchronise les utilisateurs et les licences associées |

#### Arguments
Ce type peut être appelé en passant deux arguments :
- `sync_data` : données à synchroniser
- `sync_options` : options de synchronisation

*sync_data*

Tableau d'objets composés de :
- information sur l'utilisateur
- information sur la licence
```
[{
  user: {                           # Utilisateur
    mail: String!,                  # Adresse mail
    firstname: String!,             # Prénom
    lastname: String!               # Nom
  },
  license: {                        # Licence
    license_template_code: String!, # mdb_helios.license_template.code
    helios_role: String!,           # HELIOS_INSTANCE_MANAGER ou HELIOS_INSTANCE_USER
    dates: [{                       # Dates de licence
      start_at: Date!,              # Date de début
      close_at: Date                # Date de fin
    }, (...)]
  }
}, (...)]
```

*sync_options*

Objet vide pour le moment.

```json
{}
```

#### Comportement
Cette requête peut être envoyée à intervalles réguliers (par exemple toutes les nuits) pour synchroniser la liste des utilisateurs de l'entreprise avec PROGAP.

Lorsqu'un utilisateur a été créé et sa licence démarrée à une date précise grâce à la valeur `start_at`, cette date peut plus être modifiée et doit donc toujours être renvoyée avec la même valeur **sauf si** la valeur `start_at` est supérieure à la date du jour (si la licence n'est pas réellement démarrée).

Lorsqu'un utilisateur a été créé et sa licence terminée à une date précise grâce à la valeur `close_at`, cette date peut plus être modifiée et doit donc toujours être renvoyée avec la même valeur **sauf si** la valeur `close_at` est supérieure à la date du jour (si la licence n'est pas réellement terminée). 

#### Retourne
Ce type retourne une liste de `mdb_helios_license`.

#### Exemple
Cette requête synchronise deux utilisateurs :
- Nicolas Dupont
- Francis Rio

L'utilisateur "Nicolas Dupont" 
- sera gestionnaire de licences
- sera associé au modèle de licence dont le code `mdb_helios_license_template.code = "ETUDE"`
- est ouvert depuis le 01/01/2023

L'utilisateur "Francis Rio" 
- sera utilisateur
- sera associé au modèle de licence dont le code `mdb_helios_license_template.code = "GESTION"`
- est ouvert depuis le 01/01/2023 
- est fermé depuis  le 31/03/2023
- 
```
mutation SyncLicense {
  mdb_helios_sync_license (
    args: {
      sync_data: [{
        user: {
          mail: "n.dupont@mc3i.fr",
          firstname: "Nicolas",
          lastname: "Dupont"
        },
        license: {
          license_template_code: "ETUDE",
          helios_role: "HELIOS_INSTANCE_MANAGER",
          dates: [{
            start_at: "2023-01-01"
          }]
        }
      }, {
        user: {
          mail: "f.rio@mc3i.fr",
          firstname: "Francis",
          lastname: "Rio"
        },
        license: {
          license_template_code: "GESTION",
          helios_role: "HELIOS_INSTANCE_USER",
          dates: [{
            start_at: "2023-01-01",
            close_at: "2023-03-31"
          }]
        }
      }],
      sync_options: {}
    }
  ) {
    id
  }
}
```

## Gestion des utilisateurs 

### Utilisateurs
Le type `mdb_catalog_user` correspond à un utilisateur PROGAP. 

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_user            | Liste des utilisateurs          |
| insert_mdb_catalog_user     | Ajoute des utilisateurs         |
| update_mdb_catalog_user     | Modifie des utilisateurs        |
| delete_mdb_catalog_user     | Supprime des utilisateurs       |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| mail          | text, not null, unique        | Adresse e-mail              |
| firstname     | text, not null                | Prénom                      |
| lastname      | text, not null                | Nom                         |
| profile       | integer, not null             | 0 : Administrateur<br>1 : Utilisateur |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| user_pkey                   | `id` unique                     |
| user_mail_key               | `mail` unique                   |

#### Commentaires
Tous les utilisateurs sous licences de `mdb_helios_user` doivent être aussi ajoutés au type `mdb_catalog_user` avec le même identifiant.

Si un utilisateur sous licence n'est pas renseigné dans le type `mdb_catalog_user`, PROGAP l'ajoutera automatiquement lors de la première authentification de l'utilisateur concerné.

### Rôles
Le type `mdb_catalog_role` correspond à un rôle PROGAP. 

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_role            | Liste des rôles                 |
| insert_mdb_catalog_role     | Ajoute des rôles                |
| update_mdb_catalog_role     | Modifie des rôles               |
| delete_mdb_catalog_role     | Supprime des rôles              |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| code          | text, unique                  | Code                        |
| name          | text, not null, unique        | Désignation                 |
| type          | text, not null, readonly      | 0 : Rôle<br>1 : Rôle utilisateur |
| user_id       | uuid, readonly                | Identifiant de l'utilisateur `mdb_catalog_user.id` |
| default       | boolean                       | Rôle par défaut pour tous les utiliasteurs |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| role_pkey                   | `id` unique                     |
| role_code_key               | `code` unique                   |
| role_name_key               | `name` unique                   |

#### Commentaires
Les rôles sont utilisés ensuite pour gérer les droits d'accès des utilisateurs aux données :
- Affaires (`mdb_progap_project` via le type `mdb_progap_project_role`)
- Bibliothèques (`mdb_progap_library` via le type `mdb_progap_library_role`)
- Espaces de travail (`mdb_progap_workspace` via le type `mdb_progap_workspace_role`)

Il existe 2 types de rôles :
- Les rôles standard `type = 0` : groupes utilisateurs
- Les rôles utilisateurs `type = 1` : il en existe un pour chaque `mdb_catalog_user`. Ces rôles peuvent être utilisés pour affecter des droits utilisateurs par utilisateurs. Ces rôles sont gérés par PROGAP et sont accessibles dans l'API en lecture seule.

### Affectation
Le type `mdb_catalog_grant` associe un `mdb_catalog_user` à un `mdb_catalog_role`.

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_grant           | Liste des affectations          |
| insert_mdb_catalog_grant    | Ajoute des affectations         |
| update_mdb_catalog_grant    | Modifie des affectations        |
| delete_mdb_catalog_grant    | Supprime des affectations       |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| user_id       | uuid, not null                | Identifiant de l'utilisateur `mdb_catalog_user.id` |
| role_id       | uuid, not null                | Identifiant du rôle `mdb_catalog_role.id` |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| grant_pkey                  | `id` unique                     |
| grant_user_id_role_id_key   | `user_id` et `role_id` unique   |

## Gestion des affaires 

### Espaces de travail
Le type `mdb_progap_workspace` correspond à un espace de travail PROGAP.

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_progap_workspace        | Liste des espaces de travail    |
| insert_mdb_progap_workspace | Ajoute des espaces de travail   |
| update_mdb_progap_workspace | Modifie des espaces de travail  |
| delete_mdb_progap_workspace | Supprime des espaces de travail |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| code          | text, unique                  | Code                        |
| name          | text, not null                | Désignation                 |
| workspace_id  | uuid                          | Identifiant de l'espace de travail parent `mdb_progap_workspace.id` |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| workspace_pkey              | `id` unique                     |
| workspace_code_key          | `code` unique                   |

### Droits d'accès aux espaces de travail
Le type `mdb_progap_workspace_role` associe un `mdb_catalog_role` à un `mdb_progap_workspace`.

| Type                             | Description                                        |
|----------------------------------|----------------------------------------------------|
| mdb_progap_workspace_role        | Liste des droits d'accès aux espaces de travail    |
| insert_mdb_progap_workspace_role | Ajoute des droits d'accès aux espaces de travail   |
| update_mdb_progap_workspace_role | Modifie des droits d'accès aux espaces de travail  |
| delete_mdb_progap_workspace_role | Supprime des droits d'accès aux espaces de travail |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| workspace_id  | uuid, not null                | Identifiant de l'espace de travail `mdb_progap_workspace.id` |
| role_id       | uuid, not null                | Identifiant du rôle `mdb_catalog_role.id` |
| role_type     | integer, not null             | Type du rôle `mdb_catalog_role.type` |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| workspace_role_pkey         | `id` unique                     |
| workspace_role_workspace_id_role_id_key | `workspace_id` et `role_id` unique |

### Affaires
Le type `mdb_progap_project` correspond à une affaire PROGAP. 

| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_progap_project          | Liste des affaires              |
| insert_mdb_progap_project   | Ajoute des affaires             |
| update_mdb_progap_project   | Modifie des affaires            |
| delete_mdb_progap_project   | Supprime des affaires           |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| code          | text, unique                  | Code                        |
| name          | text, not null                | Désignation                 |
| workspace_id  | uuid                          | Identifiant de l'espace de travail `mdb_progap_workspace.id` |
| archive       | boolean, not null             | Archivée                    |

#### Gestion des droits d'accès
Les affaires sont accessibles :
- A tous les utilisateurs si le champ `workspace_id = null`
- A tous les utilisateurs de l'espace de travail concerné si le champ `workspace_id <> null`

Il se peut qu'une affaire ne soit pas accessible par un utilisateur car il n'a pas accès à l'espace de travail concerné.
Cependant, il est possibe de lui donner un accès exceptionnel à une affaire d'un espace de travail grâce au type `mdb_progap_project_role`. 

### Droits d'accès aux affaires
Le type `mdb_progap_project_role` associe un `mdb_catalog_role` à un `mdb_progap_project`.

| Type                             | Description                                        |
|----------------------------------|----------------------------------------------------|
| mdb_progap_project_role          | Liste des droits d'accès aux espaces de travail    |
| insert_mdb_progap_project_role   | Ajoute des droits d'accès aux espaces de travail   |
| update_mdb_progap_project_role   | Modifie des droits d'accès aux espaces de travail  |
| delete_mdb_progap_project_role   | Supprime des droits d'accès aux espaces de travail |

#### Champs
| Nom           | Type                          | Description                 |
|---------------|-------------------------------|-----------------------------|
| id            | uuid, not null, auto, unique  | Identifiant                 |
| project_id    | uuid, not null                | Identifiant de l'espace de travail `mdb_progap_project.id` |
| role_id       | uuid, not null                | Identifiant du rôle `mdb_catalog_role.id` |
| role_type     | integer, not null             | Type du rôle `mdb_catalog_role.type` |

#### Contraintes
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| project_role_pkey           | `id` unique                     |
| project_role_project_id_role_id_key | `project_id` et `role_id` unique |
