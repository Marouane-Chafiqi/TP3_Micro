# TP3 : Spring IoC & OCP - Stratégies d'Injection et Sélection d'Implémentations

##. Objectif

Le principe Ouvert/Fermé (OCP) dit qu'une classe doit être ouverte à l'extension mais fermée à la modification.

Dans ce TP, la classe métier `MetierImpl` doit pouvoir utiliser une base de données, un fichier ou une API sans qu'on change son code. Elle dépend uniquement de l'interface `IDao`, et c'est Spring qui choisit l'implémentation à injecter.

##. Outils utilisés

- Java 11
- Maven
- Spring Context 5.3.30
- JUnit 4.13.2
- IntelliJ IDEA

##. Structure du projet

<img width="654" height="376" alt="image" src="https://github.com/user-attachments/assets/ef8684be-c8e0-4af8-bedf-0a3f6feb1af7" />


##. Ce qu'on a fait

1. Création d'un projet Maven dans IntelliJ et ajout des dépendances dans le `pom.xml`.
2. Création de l'interface `IDao` avec une seule méthode, `getValue()`. C'est le contrat commun à toutes les sources de données.
3. Création de quatre implémentations de `IDao` :

| Classe | Nom du bean | Source simulée | Valeur |
|--------|-------------|----------------|--------|
| DaoImpl | dao | Base de données (prod) | 100.0 |
| DaoImpl2 | dao2 | Base de données (dev) | 150.0 |
| DaoFile | daoFile | Fichier local | 180.0 |
| DaoApi | daoApi | API REST externe | 220.0 |

4. Création de `MetierImpl`. Elle reçoit un `IDao` par le constructeur et retourne la valeur multipliée par 2. Cette classe ne change jamais pendant le TP.
5. Création de `AppConfig` pour activer le scan des composants et charger le fichier `app.properties`.
6. Premier lancement : Spring lève une `NoUniqueBeanDefinitionException`, car il trouve quatre beans de type `IDao` et ne sait pas lequel injecter.
7. Résolution du problème avec trois stratégies (section suivante).
8. Écriture d'un test JUnit qui vérifie le résultat avec la configuration par défaut.

##. Les trois stratégies

### Stratégie A : @Primary

La classe `DaoImplPrimary` est annotée `@Primary`, donc Spring la choisit parmi les quatre candidats. C'est simple, mais pour changer de DAO il faut modifier le code et recompiler.

Lancement : `-Dspring.profiles.active=primary`

### Stratégie B : alias avec @Bean

La classe `DaoAliasConfig` déclare un bean `@Primary` qui pointe vers `DaoApi`. On change la cible dans une seule classe de configuration, sans toucher aux DAO. Il faut quand même recompiler.

Lancement : `-Dspring.profiles.active=alias`

### Stratégie C : propriétés externes

La classe `DynamicDaoConfig` lit la propriété `dao.target` dans `app.properties` et crée le DAO correspondant :

<img width="620" height="70" alt="image" src="https://github.com/user-attachments/assets/b140c77c-e85b-41a6-a122-f2001aa25a68" />


Les valeurs possibles sont `dao`, `dao2`, `daoFile` et `daoApi`. On change de source sans recompiler. C'est la stratégie par défaut, elle ne demande aucune option de lancement.

### Pourquoi des profils

Les trois stratégies déclarent un bean `@Primary`. Si elles étaient actives en même temps, Spring échouerait. Les profils garantissent qu'une seule stratégie est active à chaque exécution.

| Lancement | Stratégie active |
|-----------|------------------|
| Sans option | C |
| `-Dspring.profiles.active=primary` | A |
| `-Dspring.profiles.active=alias` | B |

##. Résultats attendus

Le résultat est toujours la valeur du DAO multipliée par 2.

| Stratégie | Configuration | DAO injecté | Résultat |
|-----------|---------------|-------------|----------|
| A | profil primary | DaoImplPrimary | 200.0 |
| B | profil alias | DaoApi | 440.0 |
| C | dao.target=dao | DaoImpl | 200.0 |
| C | dao.target=dao2 | DaoImpl2 | 300.0 |
| C | dao.target=daoFile | DaoFile | 360.0 |
| C | dao.target=daoApi | DaoApi | 440.0 |

##. Exécution finale de Main

<img width="1201" height="387" alt="image" src="https://github.com/user-attachments/assets/7c086ad2-41c4-4225-b091-69fb34185730" />

