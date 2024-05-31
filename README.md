# TP ElasticSearch NoSQL

## Partie 1 - Installation du cluster

Pour installer mon cluster ElasticSearch, j'ai décidé de passer par docker pour plus de simplicité et une mise en oeuvre plus rapide

On retrouve donc un fichier docker compose qui détaille les différents conteneurs et donc les instances d'elasticsearch. Ce fichier utilise les variables définies dans le fichier .env

On lance le cluster avec la commande : docker compose up -d
Puis lorsque les conteneurs sont toujours lancés, on ouvre depuis un navigateur l'url suivante : localhost:5601 pour avoir accès au cluster elasticsearch

On ouvre ensuite la console pour effectuer la suite du TP

## Partie 2 - Premiers pas

### Création de l'index test01

Pour créer un index, on utilise la requête suivante : PUT /test01 qui permet de créer un index dit vierge et nous renvoie le résultat suivant :
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test01"
}


Pour vérifier que cet index a bien été créé, on utilise la requête suivante : GET /test01 qui nous renvoie ainsi :
{
  "test01": {
    "aliases": {},
    "mappings": {
      "properties": {
        "date_creation": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "quantité": {
          "type": "long"
        },
        "titre": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "test01",
        "creation_date": "1717146439766",
        "number_of_replicas": "1",
        "uuid": "LE9gtOa9Rf2MKOtA0GH0Ew",
        "version": {
          "created": "8503000"
        }
      }
    }
  }
} 

### Création d'un document dans l'index test01

Pour créer un document lié à un index précis, il faut utiliser la commande suivante : 
POST _bulk?pretty
{ "index" : { "_index" : "test01" } }
{ "titre": "Premier document", "description": "C'est la première fois que je crée un document dans ES.", "quantité": 12,"date_creation": "10-05-2024"}

Cette requête nous renvoie la réponse suivante :
{
  "errors": false,
  "took": 14,
  "items": [
    {
      "index": {
        "_index": "test01",
        "_id": "5VH0zY8BPj_3SORGxYNA",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}

Ensuite pour visualiser ce document directement, on tape la requête suivante :
GET test01/_doc/_id
où _id correspond à la valeur de la clé _id de la requête précédente.
Donc si on applique cette requête avec l'id précédent, on obtient la réponse suivante :
{
  "_index": "test01",
  "_id": "5VH0zY8BPj_3SORGxYNA",
  "_version": 1,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "titre": "Premier document",
    "description": "C'est la première fois que je crée un document dans ES.",
    "quantité": 12,
    "date_creation": "10-05-2024"
  }
}

### Afficher le mapping d'un index

Pour afficher le mapping d'un index, il faut taper cette requête dans la console : GET /test01/_mapping

Et on obtient la réponse suivante :
{
  "test01": {
    "mappings": {
      "properties": {
        "date_creation": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "description": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "quantité": {
          "type": "long"
        },
        "titre": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}

On voit donc que le mapping du champ date est de type text, ce qui n'est pas bon puisque l'on souhaite avoir un champ de type date, il n'est donc pas optimal.

Mais on ne peut pas changer de mapping après avoir inséré des données dans un index. Il faut donc recréer un nouvel index avec le bon mapping.

### Création de l'index test02

Pour créer l'index test02 avec le bon mapping, il faut utiliser la commande suivante :
PUT /test02 // Pour créer l'index
PUT /test02/_mapping
{
  "properties": {
    "date_creation": {
      "type": "date"
    }
  }
} // Pour modifier le mapping du champ date_creation en type date et non pas text

On peut ensuite rajouter le même document que précédement mais cette fois-ci avec le bon mapping :
POST _bulk?pretty
{ "index" : { "_index" : "test02" } }
{ "titre": "Premier document", "description": "C'est la première fois que je crée un document dans ES.", "quantité": 12,"date_creation": "2024-05-10"}

On obtient bien une réponse de succès d'insertion.

Pour afficher comment l'analyzer a traité le champ description, on tape la requête suivante :
POST test02/_analyze
{
  "field": "description",
  "text": "C'est la première fois que je crée un document dans ES."
}

Et on obtient le résultat suivant :
{
  "tokens": [
    {
      "token": "c'est",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "la",
      "start_offset": 6,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "première",
      "start_offset": 9,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "fois",
      "start_offset": 18,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "que",
      "start_offset": 23,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "je",
      "start_offset": 27,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "crée",
      "start_offset": 30,
      "end_offset": 34,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "un",
      "start_offset": 35,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "document",
      "start_offset": 38,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "dans",
      "start_offset": 47,
      "end_offset": 51,
      "type": "<ALPHANUM>",
      "position": 9
    },
    {
      "token": "es",
      "start_offset": 52,
      "end_offset": 54,
      "type": "<ALPHANUM>",
      "position": 10
    }
  ]
}

### Création index test03

Pour créer l'index test03 avec le même mapping que l'index test02, mais sans altérer le champ description et en supprimant les mots sans valeur ajoutée, on utilise la requête suivante :
PUT /test03
{
  "settings": {
    "analysis": {
      "filter": {
        "french_stop": {
          "type": "stop",
          "stopwords": "_french_"  // Vous pouvez spécifier une langue spécifique ou une liste de stop words personnalisée
        }
      },
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "french_stop"  // Applique le filtre de stop words
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "custom_analyzer"
      },
      "date_creation": {
        "type": "date"
      }      
    }
  }
}

### Import multiple de documents en une seule requête

Pour importer plusieurs documents en une seule requête, on utilise la commande suivante :
POST /test03/_bulk
{ "index": { "_id": "1" } }
{ "titre": "Premier document", "description": "C'est la première fois que je crée un document dans ES.", "quantité": 12, "date_creation": "2024-05-10" }
{ "index": { "_id": "2" } }
{ "titre": "Deuxième document", "description": "Ceci est un deuxième document.", "quantité": 8, "date_creation": "2024-05-11" }
{ "index": { "_id": "3" } }
{ "titre": "Troisième document", "description": "Voici le troisième document.", "quantité": 15, "date_creation": "2024-05-12" }
{ "index": { "_id": "4" } }
{ "titre": "Quatrième document", "description": "Le quatrième document est ici.", "quantité": 5, "date_creation": "2024-05-13" }

La réponse retournée est la suivante :
{
  "errors": false,
  "took": 49,
  "items": [
    {
      "index": {
        "_index": "test03",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "test03",
        "_id": "2",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "test03",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "test03",
        "_id": "4",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}

 



