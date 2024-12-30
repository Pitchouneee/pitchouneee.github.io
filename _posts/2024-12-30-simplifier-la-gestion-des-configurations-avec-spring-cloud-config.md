---
title: Simplifier la gestion des configurations avec Spring Cloud Config
author: Corentin
date: 2024-12-30 14:42:00 +0100
categories: [Blog, Tutoriel]
tags: [Spring Cloud Config, Microservices, GitHub, DevOps, Configuration Management]
render_with_liquid: false
---

# Simplifier la gestion des configurations avec Spring Cloud Config

Dans un monde de plus en plus orienté vers les architectures distribuées, gérer les configurations de 
manière efficace peut vite devenir un vrai casse-tête. Entre les multiples services, les environnements 
variés (développement, test, production) et les déploiements continus, une solution centralisée et 
robuste est essentielle. C’est là que Spring Cloud Config entre en jeu ! 
Voici pourquoi et comment l’utiliser, en mettant l’accent sur son intégration avec un gestionnaire de 
contrôle de version comme GitHub ou GitLab.

## Pourquoi centraliser la gestion des configurations ?

Dans une architecture distribuée, chaque microservice a besoin de ses propres configurations :

* Paramètres pour les bases de données, 
* Clés API pour des services tiers, 
* Variables spécifiques aux environnements.

Gérer ces configurations directement dans chaque service peut vite poser des problèmes :

* Duplication des données, 
* Incohérences entre environnements, 
* Gestion lourde lors des mises à jour.

Une solution centralisée comme Spring Cloud Config apporte :

1. **Moins de redondance** : un fichier de configuration unique et partagé.
2. **Une gestion des versions facilitée** : chaque modification est historisée dans un SCM.
3. **Des déploiements simplifiés** : les services récupèrent automatiquement leurs configurations à partir d’un serveur central.

## Spring Cloud Config en quelques mots

Spring Cloud Config propose une architecture client-serveur pour gérer les configurations :

* **Serveur Config** : centralise les fichiers de configuration dans un SCM comme Git.
* **Client Config** : permet aux services de consommer ces configurations facilement.

Avec un SCM tel que Git, cette solution devient flexible et puissante :

* Les configurations sont stockées dans un dépôt Git.
* Les modifications sont historisées et peuvent être liées à des branches ou des tags.
* Les permissions renforcent la sécurité pour les accès sensibles.

## Mise en place de Spring Cloud Config

### Structure des dépôts

Voici comment sont organisés les différents dépôts pour cet exemple :

* **Server Config** : [spring-cloud-config-server](https://github.com/Pitchouneee/spring-cloud-config-server).
* **Clients** : [spring-cloud-config-client](https://github.com/Pitchouneee/spring-cloud-config-client).
* **Configurations** : [spring-cloud-config-repo](https://github.com/Pitchouneee/spring-cloud-config-repo).

Le dépôt des configurations suit l’organisation recommandée par Spring Cloud Config, avec tous les fichiers
 à la racine :

```
spring-cloud-config-repo/
  service-a-dev.yml
  service-a-prod.yml
  service-b-dev.yml
  service-b-prod.yml
```

En plus de ces fichiers spécifiques aux services et profils, un fichier `application.yml` général est 
également présent. 
Il contient les configurations communes à tous les services et profils, par exemple :

```yaml
management:
  endpoints:
    enabled-by-default: false
    web:
      exposure:
        include: "health,info,metrics"
  endpoint:
    info:
      enabled: true
    health:
      enabled: true
    metrics:
      enabled: true
```

### 1. Configurer le serveur Spring Cloud Config

Ajoutez la dépendance Spring Cloud Config Server à votre projet Maven ou Gradle :

**Maven :**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

**Gradle :**

```groovy
implementation 'org.springframework.cloud:spring-cloud-config-server'
```

Activez le serveur Config dans votre application principale :

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

Ajoutez les détails du dépôt dans le fichier `application.yml` :

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/votre-user/config-repo
          username: votre-user
          password: votre-password
```

### 2. Configurer les clients Spring Cloud Config

Ajoutez la dépendance Spring Cloud Config Client dans les services consommateurs :

**Maven :**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

Voici un exemple de configuration pour le service `service-a` :

```yaml
server.port: 8080
spring:
  application.name: service-a
  profiles:
    active: dev
  config:
    import: optional:configserver:http://localhost:8888
```

### 3. Choisir entre `application-dev` et `application-prod`

Pour sélectionner les fichiers de configuration correspondant à un environnement spécifique, utilisez la propriété `spring.profiles.active` . Cette propriété peut être définie dans le fichier de configuration client ou en tant que variable d’environnement.

Exemple pour l’environnement de développement :

```yaml
spring:
  profiles:
    active: dev
```

Lors du démarrage du service, Spring Cloud Config cherchera le fichier correspondant, par exemple `service-a-dev.yml` . Pour l’environnement de production, il suffit de changer `dev` par `prod` :

```yaml
spring:
  profiles:
    active: prod
```

Vous pouvez également définir cette propriété via la ligne de commande lors du lancement de l’application :

```bash
java -jar service-a.jar --spring.profiles.active=prod
```

Cette flexibilité permet à chaque service d’adapter ses paramètres en fonction de l’environnement cible.

## Avantages d’une intégration avec un SCM

1. **Gestion des versions** : toutes les modifications sont historisées, permettant de revenir à une version précédente si nécessaire.
2. **Collaboration simplifiée** : il est facile de travailler sur les fichiers via des pull requests ou des merge requests.
3. **Automatisation** : des pipelines CI/CD peuvent être déclenchés en fonction des changements de configuration.
4. **Sécurité** : grâce aux permissions SCM, il est possible de contrôler qui peut modifier ou consulter les configurations.

## Conclusion

Spring Cloud Config est une solution idéale pour simplifier la gestion des configurations dans une architecture distribuée. Couplé à un SCM comme GitHub ou GitLab, il offre un système centralisé, sécurisé et collaboratif qui améliore considérablement le workflow. Que vous soyez un pro des microservices ou que vous débutiez, Spring Cloud Config peut transformer la manière de gérer les configurations. Alors, prêt à vous lancer ?
