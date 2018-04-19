# Documentation de base de données Data plateforme

## Sommaire

1. [Modèle de données](##modèle-de-données)
1. [Récapitulatif](##récapitulatif)
    1. [Eficéo Collecte](###eficéo-collecte)
    1. [Eficéo Répondants](###eficéo-répondants)
    1. [XSalto Commandes en ligne](###xsalto-commandes-en-ligne)
    1. [Plateforme Eficéo](###plateforme-eficéo)
1. [Classification informations](##rappel-sur-la-catégorisation-des-champs-d'une-fiche-client)
1. [Process de dédoublonnage](##rappel-sur-le-process-de-dédoublonnage)
1. [Déscriptif des tables](##déscriptif-des-tables)
1. [Déscriptif des API XSalto](##déscriptif-des-api-xsalto)
1. [Déscriptif des Excel Eficéo](##déscriptif-des-excel-eficéo)
1. [Les tables de la base](##-les-tables)
    1. [Table de tampon Collecte](###sphinx_eficeo_collecte)
    1. [Table de tampon répondants](###sphinx_eficeo_repondant)
    1. [Table du client](###ef_client)
    1. [Table de trace](###ef_trace_client)
    1. [Trace de satisfaction](###ef_notesatisf_trace_client)
    1. [Trace de commande](###xs_activite_client)
    1. [Segmentation de la clientèle](###segmentation_de_la_clientèle)
1. [Annexe](##annexe)
    1. [Exemple des fichiers JSON](###exemple-de-fichier-json-récupéré-xsalto)
    1. [JSON commande](####exemple-de-commande)
    1. [JSON produit](####exemple-de-produit)

## Modèle de données

(06/04/2018)

![alt text](https://docs.google.com/uc?id=10aPH8KVMPEJdWv5Uy70TiBWlbvHa3jfa "Diagramme BDD")
**/!\ Atention/!\ :**
Le modèle de donnée ainsi est très variant. En effet, sequentiel est en cours de développement dessus.

## Récapitulatif

Tout d'abord, voici une liste récapitulant l'ensemble des sources d'approvisionnement de données au **03/04/2018** :

* ### Eficéo : Collecte
  * **Quoi ?** Informations récupérés par les stations de ski
  * **Comment** ? Transfert via un fichier *.csv* synchronisé
  * **Quand** ? Une fois par semaine
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_. Update/Create d'une fiche + ajout d'une trace de passage.
  * **Table tampon de synchonisation :** *sphinx_eficeo_collecte*
* ### Eficéo : Répondants
  * **Quoi ?** Informations des personnes ayant répondu aux questionnaires d'Eficéo
  * **Comment** ? Transfert via un fichier *.csv* synchronisé
  * **Quand** ? Une fois par jour
  * **Impacts BDD** ? Tables _*ef_client*_, _*ef_trace_client*_, _*ef_notesatisf_trace_client*_. Update/Create d'une fiche + ajout d'une trace de passage + ajout d'une répones questionnaire.
  * **Table tampon de synchonisation :** *sphinx_eficeo_repondant*
* ### XSalto : Commandes en ligne
  * **Quoi ?** Correspond aux informations des personnes ayant effectué une commande en ligne
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

## Rappel sur la catégorisation des champs d'une fiche client

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
> Remarque, il est possible de trier les données par date de commande (Date) ou par date de modification (lst_upd). Et on ne traite que la commande si celle-ci s'est bien effectuée : Statut == 'Close'

* XSalto a également fourni une web api permettant de récupérer les produits mis en vente (forfaits). Voici un schéma représentant ce que renvoie l'API (Exemple avec un produit) :

![alt text](https://docs.google.com/uc?id=1wgHr_UuAHXFnTxt9nR6K99Ta2vwG48TD "Schéma représentant le JSON d'un produit")
Voir annexe pour le json.

Cette api sert à retrouver le produit commandé (ligne commande). Elle permet de bien récupérer le bon produit. En effet, si une personne commande un forfait en ligne version russe, il faut retrouver le produit grâce a l'id (sinon label du forfait en russe ... ). Sert à savoir si lors de la commande il y a eu un forfait avec le mot *"enfant"* par exemple.

## Déscriptif des Excel Eficéo

Eficéo possède plusieurs Excels permettant d'apporter des données à la base :

Un Excel avec une base de **collecte**. Celle-ci contient toutes les adresses mail fournis par les stations. Voici l'exemple d'une ligne du fichier :

Domaine|Saison|Email|Point_Collecte|Semaine|Periode|Langue|Type_Visiteur
-------|------|-----|--------------|-------|-------|------|-------------
Avoriaz|Hiver 2018|pgert@yopmail.fr|Vente en ligne|Semaine 52|Période 1| Anglais| En séjour

Les type de visiteur peuvent être les suivants :

* En séjour
* Propriétaire de résidence secondaire
* Venu pour la journée depuis le domicile

Un Excel avec une base de **répondants**. Celle-ci contient toutes les informations sur les personnes ayant répondu aux questionnaire Eficeo. Ces personnes sont issues de la base de **collecte**. Voici un exemple d'une ligne du fichier (en collone car trop long) :

Champ|Valeur
-----|------
DOMAINE|Avoriaz
SAISON|Exemple : Hiver 2018
ESPACES_LUDIQUES_FREQUENTES_AV|espace1;espace2;espaceN
NOTE_SATISFACTION_GLOBALE_DOMAINE|[0-10]
SUGGESTIONS_DOMAINE|Exemple : Débit des télésièges
FIDELITE_STATION | Exemple : Vous venez tous les ans
RECOMMANDATION_STATION |[0-10]
TYPE_GROUPE| Exemple : Entre amis
NB_PERSONNES_BUDGET| Exemple : 4
SEXE_MEMBRE_1| Exemple : Un homme
AGE_MEMBRE_1| Exemple : 19
PRATIQUE_MEMBRE_1|Exemple : Tous les jours
NB_NUITS_BUDGET_SEJOUR|Exemple : 90
ENSOLEILLEMENT| Exemple : Très rare
ENNEIGEMENT| Excellente
DATE_RESERVATION_LIEU_SEJOUR|Exemple : En septembre
FREQUENTATION_ESTIVALE|Exemple : Oui
NOTE_SATISFACTION_GLOBALE_SEJOUR| [0-10]
SUGGESTIONS_SEJOUR|Exemple : Que le jacuzii d’aquariaz fonctionne grrr
PAYS_RESIDENCE| Exemple : France
PAYS_RESIDENCE_Autre| Uniquement dispo si PAYS_RESIDENCE = Autre pays
DEPARTEMENT| Exemple : 75
MAIL | michougert@yopmail.fr
NOM | Gertner
PRENOM | Michel
ADRESSE | 17 avenue d'Aléry
CODE_POSTAL|74000
VILLE|Annecy
TELEPHONE |0606060606
CNIL_INFORMATIONS | Booléen oui/non
CNIL_OFFRES_PROMOTIONNELLES |Booléen oui/non
CNIL_TENDANCESSKI|Booléen oui/non
POINT_COLLECTE| Exemple : Les Prodains
SEMAINE | Semaine 50
EMAIL|pierre@yopmail.fr
PERIODE|Période 1
PRESENCE_JEUNES_ENFANTS_BUDGET|Booléen oui/non
PRESENCE_ENFANTS_BUDGET|Booléen oui/non
PRESENCE_ADOLESCENTS_BUDGET|Booléen oui/non
GROUPE_AVEC_ENFANTS_ADOLESCENTS|Avec/sans enfant
GROUPE_AVEC_ENFANTS|Avec/sans enfant
PRESENCE_NON_SKIEURS|Booléen oui/non
SCORE_GLOBAL_CAISSES_AV|[0-10]
SCORE_GLOBAL_VE_AV|[0-10]
SCORE_GLOBAL_RM_AV|[0-10]
SCORE_GLOBAL_PISTES_AV|[0-10]
SCORE_GLOBAL_ACCUEIL_AV|[0-10]
LANG_SAISIE| Exemple : Fr
DATE_ENREG|Date de la saisie
CLE|Exemple : CFS-NICJURRUNU
INTENTIONS_RETOUR|Exemple : Oui
MODE_VISITE_2|Exemple : En séjour
CLIENT_INTERMEDIE|Booléen oui/non
CLIENT_HEBERGE_GRATUIT|Booléen oui/non

## Les tables

### sphinx_eficeo_collecte

![alt text](https://docs.google.com/uc?id=1YXcIJU1EYIxkZzTErgRJ6qLTVmcx4XR7 "sphinx_eficeo_collecte")

Cette table est directement allimentée par le **Excel Eficéo de collecte** : Chaques lignes du excel possède une occurence dans la table. (Sauf les champs calculés ainsi que STATUT).
Cette table sert de tampon de synchronisation. Le fichier excel est récupéré par sequentiel grâce à un cron et le fichier est ensuite inséré ligne par ligne dans la base de données.

### sphinx_eficeo_repondant

![alt text](https://docs.google.com/uc?id=1dwzeYCuyqUU27yqxSpqldDfgM-4aLCK5 "sphinx_eficeo_repondant")

Cette table est directement allimentée par le **Excel Eficeo des répondants**  : Chaques lignes du excel possède une occurence dans la table.
Cette table sert de tampon de synchronisation. Le fichier excel est récupéré par sequentiel grâce à un cron et le fichier est ensuite inséré ligne par ligne dans la base de données.

### ef_client

![alt text](https://docs.google.com/uc?id=1bi1xkFmeL1Y-hgfD1-lMlXFa8WkquYTw "ef_client")

Correspond à un client. Elle est remplie en fonction des différentes sources : Intervention manuelle, Eficéo Collecte, Eficéo Répondants, Vente en Ligne.
Remarque : Les clés étrangères **trace_client_cnil_id** et **trace_client_delivrabilite_id** correspondent à la dernière mise à jour CNIl du profil de la personne.

Explication de quelque champs :

* trace_cnil_client : Correspond à la dernière trace qui à mis à jour le status CNIL de la personne
* cnil : Champs permettant de savoir si la personne est de type commercial ou information. Ce champ est défini en fonction des données récupérées sur la personne. Par exemple, si la personne répond qu'elle obtenir des offres commerciales alors elle aura le statut commercial. Ou si lors d'une commande de forfait elle coche la case correspondant à la newsletter alors elle aura un statut commercial.
* Il existe plusieurs champs d'email : email,email1 et email2. En effet : Lorsqu'une personne recoit un mail d'une enquete et qu'elle répond avec un mail différent alors le mail qui aura servi à répondre sur l'email principal de la personne et l'autre l'email1 par exemple.

Voici ce qu'elle représente au sein de l'application :

![alt text](https://docs.google.com/uc?id=1YeJ61cJB18w6tU7EFe3HlxQ9z_UzswZt "view client")

>Je me passe de vous présenter les tables type **ef_pays** et **ef_langue** qui correspondent à des tables de reference...

### ef_trace_client

![alt text](https://docs.google.com/uc?id=1wuLKuq6x0i0K7Rrvtv5S3YuYAtDz5FVu "ef_trace_client")

Cette table sert à historiser tout ce qui est fait sur une personne : commande chez XSalto, réponse à un formulaire d'Eficeo, mise à jour du profil ...
Elle est donc alimentée par les sources suivantes : **Excel Eficeo des répondants** ainsi que **XSalto**. (Un peu comme un)
Les types de trace peuvent (actuellement) être :

* Create
* Update

Les sources de trace peuvent être :

* sphinx_eficeo_repondant
* sphinx_eficeo_collecte
* user_eficeo_platform
* xsalto_commande

Sur la plateforme, il est possible de viusaliser 2 sortes de traces (pour le moment) : Les réponses aux **questionnaire** ainsi que les **commandes**.

Tout d'abord les traces de questionnaires.

### ef_notesatisf_trace_client

![alt text](https://docs.google.com/uc?id=1oCYy9D5VKozL_YNwfS5fqanz_aFkBklT "ef_notesatisf_trace_client")

On remarque donc que la grande majorité des informations dans la table sont issues de la table sphinx_eficeo_repondant. Aussi, cette table est liée par une clé étrangère à une trace (puisque que c'est une sorte de trace).
**A partir d'une trace donnée on peut donc retrouver son contenu : Réponse à un questionnaire ici**. Donc la trace laisée sera de type sphinx_eficeo_repondant ici.
Visuellement, on retrouve leur affichage comme ceci :

![alt text](https://docs.google.com/uc?id=1hjb38Ej-omHxiGeziKK27v-HhiuicpiD "visualisation des traces réponses")

Maintenant on va voir les commandes XSalto.

### xs_activite_client

![alt text](https://docs.google.com/uc?id=1rFEOxzEsF6XTd0PkS5o3YLfxowNpyT_p "xs_activite_client")
On observe que cette table est directemtn liée à **prx_sync_client**. En effet, on a vu que la table **prx_sync_client** servait de tampon pour n'importe quelle web api. Pour le moment, elle sert de tampon à la web API XSalto : On récupère le contenu de la réponse de l'api dans le champ data_json puis un traitement en php (Symphony en fait) est fait pour mettre les informations dans la table **xs_activite_client**.
Voici un diagramme de séquence représentant la transition entre la web api et la création de trace :

![alt text](https://docs.google.com/uc?id=182llRC6ulyfULYq4QC0PT45j6gEb5jGX "seq_api_data")

Suite à ce traitement, on voit donc qu'une trace "XSalto Commande" est générée (dource_id=4) :
![alt text](https://docs.google.com/uc?id=1mou0EqGgQNYXxUuQSk9tCe_WPvsqkHOe "ligne_de_resultats")

**Remarque 1 : La table prx_sync_client pourra permettre à sequentiel  d'utiliser d'autre web API en gardant le contenu json.**
**Remarque 2 : Le champ nbr_forfait est calculé en fonction de la taille du tableau data[] (situé en dessous de relationships/lines). Aussi, le champ pres_enfant est calculé (bool) en fonction du type de porduits qui dont commandés : Si le libellé d'origine contient "enfant" alors c'est true sinon false.**
**Remarque 3 : (Après MaJ) : A partir d'une trace donnée on peut donc retrouver le contenu de la commande (clé étrangère).**
Visuellement, on retrouve leur affichage comme ceci :
![alt text](https://docs.google.com/uc?id=1nr79zehCf0EtkJ_Hm_xAcBQzoZSiBJaH "affichage_resultats")

### Segmentation_de_la_clientèle

L'application permet aux utilisateurs d'éffectuer des segments sur leur clientèle grâce à différents filtres (âge,pays,fidèlité, ...). Un segment est défini par un nom et est composé de filtres.
Pour permettre cela voici comment les segments sont faits dans la base  :

![alt text](https://docs.google.com/uc?id=1pbr5Dd4xdR58RY7wywYMvGbK6GLo2BpT "segm_tables")

Voici un exemple de lignes que l'on peut retrouver dans la table des filtres :

![alt text](https://docs.google.com/uc?id=1Tm_k__l711OrYnBx2xTMUz127vp2D4ui "segm_tables")

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

@PierreGertner
