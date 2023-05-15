[API](..) / Schéma / Liste complète des types

## Sommaire
- Données utilisateurs (`mdb_catalog_*`)
  - [Utilisateurs](#utilisateurs)
  - [Rôles](#rôles)
  - [Affectation](#affectation)

## Données utilisateurs 

### Utilisateurs
Le type `mdb_catalog_user` correspond à un utilisateur PROGAP.

#### Types
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_user            | Liste des utilisateurs          |
| insert_mdb_catalog_user     | Ajoute des utilisateurs         |
| update_mdb_catalog_user     | Modifie des utilisateurs        |
| delete_mdb_catalog_user     | Supprime des utilisateurs       |

### Rôles
Le type `mdb_catalog_role` correspond à un rôle PROGAP.

#### Types
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_role            | Liste des rôles                 |
| insert_mdb_catalog_role     | Ajoute des rôles                |
| update_mdb_catalog_role     | Modifie des rôles               |
| delete_mdb_catalog_role     | Supprime des rôles              |

### Affectation
Le type `mdb_catalog_grant` associe un `mdb_catalog_user` à un `mdb_catalog_role`.

#### Types
| Type                        | Description                     |
|-----------------------------|---------------------------------|
| mdb_catalog_grant           | Liste des affectations          |
| insert_mdb_catalog_grant    | Ajoute des affectations         |
| update_mdb_catalog_grant    | Modifie des affectations        |
| delete_mdb_catalog_grant    | Supprime des affectations       |
