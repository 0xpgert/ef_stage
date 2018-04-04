# Documentation de base de données Data plateforme

## Sommaire

## Model de données

![alt text](https://docs.google.com/uc?id=10aPH8KVMPEJdWv5Uy70TiBWlbvHa3jfa "Diagramme BDD")

## Récapitulatif

Tout d'abord, voici une liste récapitulant l'ensemble des sources d'approvisionnement de données au **03/04/2018** :

* ### Eficéo : Collecte : Informations récupérés par les stations de ski
  * **Comment** ? Transfert via un fichier *.csv* synchronisé
  * **Quand** ? Une fois par semaine
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_. Update/Create d'une fiche + ajout d'une trace de passage.
  * **Table tampon de synchonisation :** *sphinx_eficeo_collecte*
* ### Eficéo : Répondants : Informations des personnes ayant répondu aux questionnaires d'Eficéo
  * **Comment** ? Transfert via un fichier *.csv* synchronisé
  * **Quand** ? Une fois par jour
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_, _*ef_notesatisf_trace_client*_. Update/Create d'une fiche + ajout d'une trace de passage + ajout d'une répones questionnaire.
  * **Table tampon de synchonisation :** *sphinx_eficeo_repondant*
* ### XSalto : Commandes en ligne : Correspond aux informations des personnes ayant effectué une commande en ligne
  * **Comment** ? Appel d'un web API. _(Décrit plus tard)_
  * **Quand** ? Une fois par jour
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_, _*xs_activite_client*_. Update/Create d'une fiche + ajout d'une trace de passage + ajout d'une commande de forfait en ligne
  * **Table tampon de synchonisation :** *prx_sync_client*
* ### Plateforme Eficéo
  * **Comment** ? Via la plateforme
  * **Quand** ? Quand l'utilisateur effectue une action engendrant une modification.
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_. Update/Create d'une fiche + ajout d'une trace de passage.
  * **Table tampon de synchonisation:** *Aucune*
> Il manque juste TeamAcess, qui fournira les données concernant les remontées mecaniques.

## Rappel sur la catégorisation des champs d'une fiche client _(cf Fichier des règles de la fiche client )_

Sigle (type de champ) | Signification | Exemples
------|---------------|----------
CIP | Champ d'Identification Primaire |email, nom, prénom, sexe
CIS | Champ d'Identité Secondaire|Année de naissance, adresse mail 2, téléphone ...
CSCNIL | Champ status CNIL | Date de mise à jour du statut de délivrabilité ...
CSA | Champ de statut autre | Fidelité, présence d'enfant ...
CT | Champ Trace | Semaine venues Station, Type d'activité, mode de visites lors des venues ...
CAJS | Champ d'Activité Journée Ski | Date de la journée, saison, dénivelé ...
CAAF | Champ d'Activité Achat Forfait | Forfait saison, saison, mode de paiement, montant total de l'achat ...
CACM | Champ d'Activité Campagne Mail | Date de l'envoi du mail, mom de la campagne, date de réaction, désabonnement ...
CAR | Champ d'Activité Réclamation | Réclamation, origine, date de la réclamation ...
CAS | Champ d'Activité Satisfaction | Semaine venue station, satisfaction domaine, satisfaction séjour, intention de retour ...

## Rappel sur le process de dédoublonnage

![alt text](https://docs.google.com/uc?id=1bAGirVruRD7qb-zZryr08sYYBBBLFItR "Diagramme d'activité dédoublonnage")

## Déscriptif des tables

## Déscriptif des API XSalto

* **XSalto** a fourni une web API qui permet de récupérer les informations qui concernent les commandes passées le jour même en format **json**. Voici un schéma représentant ce que renvoie l'api (Exemple avec une commande) :

![alt text](https://docs.google.com/uc?id=1eoLICl0l4qBx7zrXyh4HI_l6HnmRhJJV "Schéma représentant le JSON d'une commade")
Voir annexe pour le json.

> On remarque donc que l'on récupère des informations permettant de créer une fiche client. En effet, plusieurs CIP sont présents : email, nom, prenom. De plus, dans la branche **lines** on récupère 2 éléments. Ce sont des produits : ```"Type":"Product"``` .Aussi, on peut observer que se sont des forfaits : ```"Type":"forf"```. On récupère également le prix des produits ainsi que le label.
> Remarque, il est possible de trier les données par date de commande (Date) ou par date de modification (lst_upd).

* XSalto a également fourni une web api permettant de récupérer les produits mis en vente (forfaits). Voici un schéma représentant ce que renvoie l'API (Exemple avec un produit) :

![alt text](https://docs.google.com/uc?id=1wgHr_UuAHXFnTxt9nR6K99Ta2vwG48TD "Schéma représentant le JSON d'un produit")
Voir annexe pour le json.

## Déscriptif des Excel Eficéo

Eficéo possède plusieurs Excels permettant d'apporter des données à la base :

* Un Excel avec une base de **collecte**. Celle-ci contient toutes les adresses mail fournis par les stations. Voici l'exemple d'une ligne du fichier :

* Un Excel avec une base de **répondants**. Celle-ci contient toutes les informations sur les personnes ayant répondus aux questionnaire Eficeo. Ces personnes sont issues de la base de **collecte**. Ainsi, il est possbble de savoir si une personne à répondu ou non. Voici l'exemple d'une ligne du fichier :

## Annexe

### Exemple de fichier json récupéré XSalto

#### Exemple de commande

```json
{
    "lst_upd":"2018-04-04 09:19:05",
    "attributes":{
       "Season":{
          "name":"2017/2018",
          "id":"2017-2018"
       },
       "Reference":"A20180404090007",
       "Date":"2018-04-04 09:17:50",
       "Type":"R",
       "LastName":"LastNameExample",
       "FirstName":"FisrtNameExample",
       "FirstDay":"2018-04-04",
       "Status":"Close",
       "PaymentMode":"Carte bancaire",
       "Total":86,
       "Lang":"FR",
       "EMail":"example@ex.io",
       "Phone":null,
       "Mobile":"0606060606",
       "Address1":null,
       "Address2":null,
       "Address3":null,
       "ZipCode":null,
       "City":null,
       "Country":"France",
       "DeliveryLastName":"Example",
       "DeliveryFirstName":"Example",
       "DeliveryAddress1":null,
       "DeliveryAddress2":null,
       "DeliveryAddress3":null,
       "DeliveryZipCode":null,
       "DeliveryCity":null,
       "DeliveryCountry":"France",
       "FinalCustomer":null,
       "ProOrder":false,
       "FinalCustomerEmail":null,
       "FreeParking":false,
       "Group":null,
       "NewsLetter":false
    },
    "relationships":{
       "lines":{
          "data":[
             {
                "lst_upd":"2018-04-04 09:19:05",
                "attributes":{
                   "Reference":"040917251/0010",
                   "Label":"1 jour Adulte Avoriaz",
                   "Type":"forf",
                   "Date":"2018-04-04",
                   "WTP":"N582A039-0Q0-YFX",
                   "LastName":"LNameExample p1",
                   "FirstName":"FNameExample p1",
                   "DateOfBirth":"1986-04-14",
                   "Price":43,
                   "OriginalPrice":null,
                   "SpecialOffer":null
                },
                "relationships":{
                   "Offer":{
                      "data":{
                         "type":"Offer",
                         "id":"4zes4ttick68"
                      }
                   },
                   "Product":{
                      "data":{
                         "type":"Product",
                         "id":"afc54ret12jf"
                      }
                   }
                },
                "type":"lines",
                "id":"kx55ebrm9wae"
             },
             {
                "lst_upd":"2018-04-04 09:19:05",
                "attributes":{
                   "Reference":"040917252/0010",
                   "Label":"1 jour Adulte Avoriaz",
                   "Type":"forf",
                   "Date":"2018-04-04",
                   "WTP":"N582A039-0PS-GFX",
                   "LastName":"LNameExample p2",
                   "FirstName":"FNameExample p2",
                   "DateOfBirth":"1989-10-28",
                   "Price":43,
                   "OriginalPrice":null,
                   "SpecialOffer":null
                },
                "relationships":{
                   "Offer":{
                      "data":{
                         "type":"Offer",
                         "id":"4zes4ttick68"
                      }
                   },
                   "Product":{
                      "data":{
                         "type":"Product",
                         "id":"afc54ret12jf"
                      }
                   }
                },
                "type":"lines",
                "id":"adi5ebrm9waz"
             }
          ]
       }
    },
    "type":"orders",
    "id":"h6i5ebrmfktn"
 }
```

#### Exemple de produit

```json
{
  "lst_upd": "2018-03-31 10:58:49",
  "attributes": {
    "label": null,
    "pool": {
      "label": "Avoriaz",
      "id": "avoriaz"
    },
    "person": {
      "label": "Jeune",
      "id": "h245dqcdut0f"
    },
    "ticket": {
      "label": "3 jours",
      "id": "4jvkfm96kjt5"
    },
    "tapool": {
      "no": 1,
      "label": "Avoriaz Hiver",
      "id": "000477000001"
    },
    "taperson": {
      "no": 27,
      "label": "Pâques",
      "id": "000027"
    },
    "taticket": {
      "no": 52006,
      "label": "3 Jours F (Avoriaz Hiver)",
      "id": "052006001"
    }
  },
  "relationships": [],
  "type": "products",
  "id": "ni45a2bm6avd"
}
```

@Pierre Gertner
