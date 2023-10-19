[API](..) / Schéma / Référence

## Sommaire
- Gestion des utilisateurs (*en cours d'écriture*)
  - [Utilisateurs](#utilisateurs)
  - [Rôles](#rles)
  - [Affectation](#affectation)
  - [Licence](#licences)
- Gestion des affaires (*en cours d'écriture*)
  - [Espaces de travail](#espaces-de-travail)
  - [Droits d'accès aux espaces de travail](#droits-daccs-aux-espaces-de-travail)
  - [Affaires](#affaires)
  - [Droits d'accès aux affaires](#droits-daccs-aux-affaires)
- Données calculées (*en cours d'écriture*)
  - [Données calculées de bibliothèque](#donnes-calcules-de-bibliothque)
  - [Données calculées de devis](#donnes-calcules-de-devis)

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


### Licences

Deux types existent pour synchroniser les utilisateurs et les licences :
| Type                            | Description                     |
|---------------------------------|---------------------------------|
| mdb_catalog_sync_start_license  | Ajoute ou modifie des licences  |
| mdb_catalog_sync_close_license  | Supprime des licences           |

#### Ajouter ou modifier des licences
Pour ajouter des licences, il suffit d'envoyer la liste des utilisateurs concernés ainsi que le code de modèle de licence configuré avec M&C 3i.

Par exemple, cette requête ajoute (ou modifie) la licence de deux utilisateurs :
- Nicolas Dupont
- Francis Rio

L'utilisateur "Nicolas Dupont" 
- sera désormais gestionnaire de licences
- sera désormais associé au modèle de licence dont le code `mdb_helios_license_template.code = "ETUDE"`

L'utilisateur "Francis Rio" 
- sera désormais utilisateur
- sera désormais associé au modèle de licence dont le code `mdb_helios_license_template.code = "GESTION"`

```
mutation StartLicense {
  mdb_catalog_sync_start_license(
    objects: [{
    	user_mail: "n.dupont@mc3i.fr", 
      user_firstname: "Nicolas", 
      user_lastname: "Dupont", 
      helios_role: "HELIOS_INSTANCE_MANAGER", 
      license_template_code: "ETUDE"
    }, {
      user_mail: "f.rio@mc3i.fr", 
      user_firstname: "Francis", 
      user_lastname: "Rio", 
      helios_role: "HELIOS_INSTANCE_USER", 
      license_template_code: "GESTION"
    }], 
    options: {}
  ) {
    id
  }
}
```

Les mouvements de licences effectués sur un utilisateur (passage d'un modèle de licence à un autre) peuvent mettre jusqu'à 5 minutes à se propager dans PROGAP web.

Lors d'un passage d'un modèle de licence à un autre, le précédent modèle de licence sera facturé jusqu'à la veille du changement et le nouveau modèle sera facturé à partir du jour du changement.

#### Supprimer des licences
Pour supprimer des licences, il suffit d'envoyer la liste des utilisateurs concernés ainsi que le code de modèle de licence configuré avec M&C 3i.

Par exemple, cette requête ajoute (ou modifie) la licence de deux utilisateurs :
- Nicolas Dupont
- Francis Rio

```
mutation CloseLicense {
  mdb_catalog_sync_close_license(
    objects: [{
    	user_mail: "n.dupont@mc3i.fr"
    }, {
      user_mail: "f.rio@mc3i.fr"
    }], 
    options: {}
  ) {
    id
  }
}
```

Une licence supprimée sera facturé jusqu'à la veille de sa suppression.

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

## Données calculées

Certaines données de PROGAP ne sont pas stockées mais sont calculées en temps réel (par exemple le prix
total d'un devis est calculé en temps réel lorsque l'interface utilisateur en a besoin).

### Données calculées de bibliothèque
Le type `mui_progap_library_global_info` permet de récupérer toutes les données calculées en temps réel
relatives à une bibliothèque.

Ces donnée sont généralement : 
- Les valeurs de la bibliothèque et les informations associées à un devis (tâches, zones ...)
- Les ratios de la bibliothèque et les informations associées à un devis (tâches, zones ...)

Depuis le type `mui_progap_library_global_info`, il est possible de récupérer les valeurs et les ratios 
des informations suivantes :
- De la bibliothèque
- Des articles
- Des sous-détails
- Des prix unitaires par version de bibliothèque


**Exemple de reqûete**

La requête ci-dessous récupère les données calculées des marchés d'une bibliothèque dont l'identifiant de la table `mdb_progap_library` est `14d87084-ca0b-4074-abb0-d2b306db2919`.

```
# Requête

query {
  mui_progap_library_global_info(args: {

    # L'identifiant de la bibliothèque
    library_id: "14d87084-ca0b-4074-abb0-d2b306db2919"

    # Options des valeurs à récupérer
    options: {

      # Articles
      items: true
    }
  }) {
    info
  }
}
```

```jsonc
// Réponse

{
  "data": {
    "mui_progap_library_global_info": [
      {

        // Données calculées de bibliothèque
        "info": {

          // Liste des articles
          "items": [
            {
              // Identifiant de l'article
              "id": "80aa10e6-2fbf-44ff-9895-2e3ac10dae02",

              // Code de l'article
              "code": "407AGC20",

              // Prix unitaire dans la devise de la bibliothèque
              "cost_a": 41.57,

              // Prix unitaire dans la devise de l'article
              "cost_b": 41.57
            }, {
              // ... autres articles
            }
          ]
        }
      }
    ]
  }
}
```

#### Options détaillées de requête

```
# Requête

query {
  mui_progap_library_global_info(args: {

    # L'identifiant de la bibliothèque
    library_id: "14d87084-ca0b-4074-abb0-d2b306db2919"

    # Options des valeurs à récupérer
    options: {

      # Articles
      items: true

      # Articles détaillés
      items: {

        # Compteurs
        counters: true
      }

      # Prix unitaires par version de bibliothèque
      virtual_components: true
    }
  }) {
    info
  }
}
```

#### Valeurs détaillées de réponse

```jsonc
// Réponse

{
  "data": {
    "mui_progap_library_global_info": [
      {

        // Données calculées de bibliothèque
        "info": {

          // Liste des articles
          "items": [
            {
              // Identifiant de l'article
              "id": "80aa10e6-2fbf-44ff-9895-2e3ac10dae02",

              // Code de l'article
              "code": "407AGC20",

              // Prix unitaire dans la devise de la bibliothèque
              "cost_a": 41.57,

              // Prix unitaire dans la devise de l'article
              "cost_b": 41.57,

              // Compteurs
              "counters": [
                {

                  // Identifiant de la famille analytique (mdb_progap_section.id)
                  "counter_id": "d931a40e-8778-4495-b6b4-5308a4a1777a",

                  // Prix unitaire dans la devise de la bibliothèque
                  "cost_a": 27.6,

                  // Prix unitaire dans la devise de l'article
                  "cost_b": 27.6,

                  // Quantité
                  "quantity": 1.2
                }
              ]
            }, {
              // ... autres articles
            }
          ],

          // Liste des prix unitaires par version de bibliothèque
          "virtual_components": [
            {

              // Identifiant de l'article
              "item_id": "b6c72b04-b220-4375-8aa1-4882baebb051",

              // Identifiant de la version de bibliothèque (mdb_progap_library_version.id)
              "library_version_id": "21b9e21f-c9ab-46a6-828c-97813ed18ee7",

              // Prix unitaire dans la monnaie de la bibliothèque
              "cost_a": 131.36,

              // Prix unitaire dans la monnaie de la version 
              "cost_b": 131.36
            }, {
              // ... autres prix unitaires par version de bibliothèque
            }
          ]
        }
      }
    ]
  }
}
```

### Données calculées de devis
Le type `mui_progap_estimate_global_info` permet de récupérer toutes les données calculées en temps réel
relatives à un devis.

Ces donnée sont généralement : 
- Les valeurs du devis et les informations associées à un devis (tâches, zones ...)
- Les ratios du devis et les informations associées à un devis (tâches, zones ...)

Depuis le type `mui_progap_estimate_global_info`, il est possible de récupérer les valeurs et les ratios 
des informations suivantes :
- Du devis 
- Des zones 
- Des lots
- Des budgets
- Des marchés
- Des familles analytiques 
- Des colonnes de coefficients de vente
- Des ventilations
- Des tâches
- Des familles de vente
- Des variables
- Des codes techniques
- Des quantités par zones

**Exemple de reqûete**

La requête ci-dessous récupère les données calculées des marchés d'un devis dont l'identifiant de la table `mdb_progap_estimate` est `d9eeb1a2-2b46-4e39-ae2c-b5dfca0ff5ac`.

```
# Requête

query {
  mui_progap_estimate_global_info(args: {

    # L'identifiant du devis
    estimate_id: "d9eeb1a2-2b46-4e39-ae2c-b5dfca0ff5ac"

    # Options des valeurs à récupérer
    options: {

      # Marchés
      contracts: true
    }
  }) {
    info
  }
}
```

```jsonc
// Réponse

{
  "data": {
    "mui_progap_estimate_global_info": [
      {

        // Données calculées du devis
        "info": {

          // Valeur déboursé
          "cost": 362714.263,

          // Valeur vente
          "sold": 520000,

          // Valeur ventilation
          "spread": 520000,

          // Valeur devis commercial
          "quotation": 520010.168,

          // K vente
          "k_sold": 1.43364,

          // K ventilation
          "k_spread": 1.43364,

          // K devis commercial
          "k_quotation": 1.43366,

          // Liste des marchés
          "contracts": [
            {
              // Rappel de l'identifiant du devis
              "estimate_id": "d9eeb1a2-2b46-4e39-ae2c-b5dfca0ff5ac",

              // Identifiant du marché
              "id": "719d9ca1-5605-4c52-832c-c1137f0e7d0a",

              // Valeur déboursé
              "cost": 362714.263,

              // Valeur vente
              "sold_calculated": 520000,

              // K vente
              "k_sold": 1.43364,

              // % / déboursé du devis
              "cost_proportion": 1,

              // % / vente du devis
              "sold_proportion": 1
            }, {
              // ... autres marchés
            }
          ]
        }
      }
    ]
  }
}
```

#### Options détaillées de requête

```
# Requête

query {
  mui_progap_estimate_global_info (args: {
    
    # L'identifiant du devis
    estimate_id: "d9eeb1a2-2b46-4e39-ae2c-b5dfca0ff5ac"

    # Options des valeurs à récupérer
    options: {

      # Zones
      areas: true

      # Zones détaillées
      areas: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Lots
      batches: true

      # Lots détaillés
      batches: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Budgets
      budgets: true

      # Budgets détaillés
      budgets: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Marchés
      contracts: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Familles analytiques
      sections: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Colonnes de coefficients de vente
      sells: true

      # Ventilations
      spreads: true

      # Tâches
      tasks: true

      # Tâches détaillées
      tasks: {

          # Lots
      		batches: true

          # Compteurs
      		counters: true

          # Consultations
      		consultations: true

          # Articles
      		items: true

          # Variables
      		variables: true				
      }

      # Familles de vente
      trades: true

      # Familles de vente détaillées
      trades: {

        # Compteurs
      	counters: true

        # Variables
      	variables: true				
      }

      # Variables
      variables: true

      # Variables détaillées
      variables: {

        # Compteurs
      	counters: true
      }

      # Codes techniques
      zones: true

      # Codes techniques détaillés
      zones: {

          # Compteurs
      		counters: true

          # Variables
      		variables: true				
      }

      # Quantités par zones
      virtual_forecasts: true
    }
  }) {
    info
  }
}
```

#### Valeurs détaillées de réponses

La réponse détaillée ci-dessous (réalisée par zone) est identique pour :
- Des lots
- Des budgets
- Des marchés
- Des familles analytiques 
- Des colonnes de coefficients de vente
- Des ventilations
- Des tâches
- Des familles de vente
- Des variables
- Des codes techniques
- Des quantités par zones

```jsonc
{
  "data": {
    "mui_progap_estimate_global_info": [
      {

        // Valeurs de devis
        "info": {
          
          // Valeur déboursé
          "cost": 362714.263,

          // Valeur vente
          "sold": 520000,

          // Valeur ventilation
          "spread": 520000,

          // Valeur devis commercial
          "quotation": 520010.168,

          // K vente
          "k_sold": 1.43364,

          // K ventilation
          "k_spread": 1.43364,

          // K devis commercial
          "k_quotation": 1.43366,

          // Zones
          "areas": [
            {

              // Identifiant de la zone
              "id": "796a7e89-2db8-49ed-8973-eda3d29a9240",

              // Valeur déboursé
              "cost": 362714.263,

              // Valeur vente
              "sold": 520000,

              // Valeur devis commercial
              "quotation": 520010.168,

              // K vente
              "k_sold": 1.43364,

              // % / valeur déboursé du devis
              "cost_proportion": 1,

              // % / valeur vente du devis
              "sold_proportion": 1,

              // Compteurs
              "counters": [
                {
                  // Identifiant de la famille analytique associée
                  "counter_id": "d931a40e-8778-4495-b6b4-5308a4a1777a",

                  // Quantité déboursé
                  "quantity": 4681.978,

                  // Valeur déboursé
                  "cost": 107685.494,

                  // Valeur vente
                  "sold": 159070.801
                }, {
                  //... autres compteurs
                }
              ],

              // Variables
              "variables": [
                {

                  // Identifiant de la variable
                  "variable_id": "604fd744-cf8b-4b21-91ae-c50c5f970f29",

                  // Déboursé unitaire (valeur déboursé de la zone / quantité de la variable) 
                  "unit_cost": 362.714,

                  // Vente unitaire (valeur vente de la zone / quantité de la variable)
                  "unit_sold": 520
                }, {
                  // ... autres variables
                }
              ]
            }, {
              // ... autres zones
            }
          ]
        }
      }
    ]
  }
}
```
