[API](..) / Schéma / Généralités

## Types 
Pour faciliter la lecture et la compréhention du schéma de l'API GraphQL PROGAP web, les différents types (Queries, Mutations) sont codifiés selon les patterns suivants.

### Queries 
| Nom                     | Description                                             |
|-------------------------|---------------------------------------------------------|
| `mdb_batichiffrage_*`   | Données de l'interface avec BatiChiffrage               |
| `mdb_catalog_*`         | Données utilisateurs (id, nom, prénom ...)              |
| `mdb_helios_*`          | Données de licences et de facturation                   |
| `mdb_progap_*`          | Données métier PROGAP web                               |
| `mdb_progapwin_*`       | Données de l'interface avec l'application PROGAP win    |
| `mui_progap_*`          | Vues de données métier PROGAP web                       |

### Mutations
| Nom                     | Description
|-------------------------|---------------------------------------------------------|
| `insert_*`              | Ajoute des enregistrements                              |
| `update_*`              | Modifie des enregistrements                             |
| `delete_*`              | Supprime des enregistrements                            |

### Exemples
- `mdb_progap_project` : récupère les affaires
- `insert_mdb_progap_project` : ajoute des affaires
- `update_mdb_progap_project` : modifie des affaires
- `delete_mdb_progap_project` : supprime des affaires
