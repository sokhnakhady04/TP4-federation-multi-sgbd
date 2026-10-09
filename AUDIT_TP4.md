# AUDIT_TP4 --- Fédération multi-SGBD avec PySpark

## 1. Objet

Ce document présente l'audit d'intégrité inter-SGBD réalisé dans le TP4
de fédération multi-SGBD avec PySpark.

Les quatre sources utilisées sont :

-   **MySQL** : `ecommerce_crm.clients`
-   **PostgreSQL** : `ecommerce_catalogue.produits`
-   **Oracle** : `FREEPDB1 / SPARK_USER.COMMANDES`
-   **SQL Server** : `ecommerce_paiements.dbo.paiements`

L'audit a été réalisé avec des `LEFT ANTI JOIN` afin d'identifier les
enregistrements présents dans une source mais sans correspondance dans
la source de référence.

------------------------------------------------------------------------

## 2. Résultat de l'audit d'intégrité

### Point important sur l'énoncé du TP

Le point 12 du notebook demande de documenter **« les 3 anomalies
d'intégrité trouvées en partie 7 »**.

Cependant, dans l'exécution effectivement réalisée, l'audit du point 7 a
produit :

-   commandes sans client : **0**
-   commandes sans produit : **0**
-   paiements orphelins : **0**

Par conséquent, **aucune anomalie d'intégrité n'a été trouvée dans les
données réellement chargées**.

Il serait incorrect de fabriquer trois anomalies ou d'inventer des
identifiants pour satisfaire littéralement cette consigne. Le présent
document conserve donc les résultats réellement observés.

### 2.1 Commandes sans client

**Identifiants trouvés :** aucun.

-   **Table source :** `COMMANDES`
-   **SGBD source :** Oracle
-   **Table de référence :** `clients`
-   **SGBD de référence :** MySQL
-   **Résultat :** `0`

**Impact métier :** aucune commande orpheline au niveau du client n'a
été détectée dans les données testées ; chaque commande possède une
correspondance avec un client.

### 2.2 Commandes sans produit

**Identifiants trouvés :** aucun.

-   **Table source :** `COMMANDES`
-   **SGBD source :** Oracle
-   **Table de référence :** `produits`
-   **SGBD de référence :** PostgreSQL
-   **Résultat :** `0`

**Impact métier :** aucune commande ne référence un produit absent du
catalogue PostgreSQL dans les données testées.

### 2.3 Paiements orphelins

**Identifiants trouvés :** aucun.

-   **Table source :** `paiements`
-   **SGBD source :** SQL Server
-   **Table de référence :** `COMMANDES`
-   **SGBD de référence :** Oracle
-   **Résultat :** `0`

**Impact métier :** aucun paiement ne référence une commande inexistante
dans Oracle dans les données testées.

------------------------------------------------------------------------

## 3. Contrôle complémentaire --- commandes PAYEE sans paiement réussi

Un contrôle complémentaire a été réalisé afin d'identifier les commandes
ayant le statut `PAYEE` mais ne possédant aucun paiement avec le statut
`PAYE`.

**Résultat : `0`**

Aucune commande `PAYEE` sans paiement réussi n'a été détectée.

------------------------------------------------------------------------

## 4. Tableau de relevés final

  Mesure                                 Valeur
  ------------------------------ --------------
  nb_clients (MySQL)                         12
  nb_produits (PostgreSQL)                   10
  nb_commandes (Oracle)                      30
  nb_paiements (SQL Server)                  30
  jointure commandes-clients                 30
  jointure 3 tables                          30
  vue federee (4 SGBD)                       22
  commandes sans client                       0
  commandes sans produit                      0
  paiements orphelins                         0
  livrees sans paiement reussi                0
  CA federe total (FCFA)           5 685 000.00

------------------------------------------------------------------------

## 5. Interprétation

L'audit montre que les quatre sources peuvent être rapprochées sans
détecter d'anomalie référentielle dans le jeu de données utilisé.

La fédération finale contient **22 lignes**, issues des commandes
répondant aux critères de la vue fédérée, pour un chiffre d'affaires
total de **5 685 000 FCFA**.

L'absence d'anomalies dans ce jeu de test ne signifie pas qu'une
architecture réelle multi-SGBD ne nécessite pas de contrôles
d'intégrité. Au contraire, lorsque les clés étrangères ne sont pas
gérées par un SGBD unique, les contrôles applicatifs ou analytiques tels
que les anti-jointures sont nécessaires pour détecter les incohérences
entre systèmes.

------------------------------------------------------------------------

## 6. Synthèse technique

Le TP a démontré les opérations suivantes :

1.  lecture JDBC de quatre SGBD hétérogènes ;
2.  création de DataFrames Spark ;
3.  création de vues temporaires Spark SQL ;
4.  normalisation des clés entre systèmes ;
5.  jointures inter-SGBD ;
6.  contrôle d'intégrité par `LEFT ANTI JOIN` ;
7.  filtrage des commandes et paiements réussis ;
8.  calculs analytiques ;
9.  observation du pushdown JDBC ;
10. écriture de la vue fédérée dans PostgreSQL ;
11. relecture et vérification du résultat consolidé.

------------------------------------------------------------------------

## 7. Conclusion

Le contrôle d'intégrité réalisé dans le TP4 est concluant pour les
données de test utilisées : **0 commande sans client, 0 commande sans
produit et 0 paiement orphelin**.

Le résultat doit être présenté comme un **audit sans anomalie
détectée**, et non comme un audit ayant trouvé trois anomalies. Cette
distinction est importante pour conserver une restitution fidèle à
l'exécution réelle du TP.
