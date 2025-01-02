---
title: Réduire la taille de vos conteneurs Docker
author: Corentin
date: 2025-01-03 12:00:00 +0100
categories: [Conteneurisation]
tags: [Docker, Best Practices, Performance]
render_with_liquid: false
---

Dans le monde de la conteneurisation, avoir des images Docker légères, c’est un peu comme voyager avec un bagage cabine : plus rapide, plus pratique, et beaucoup moins de stress. Voici quelques astuces concrètes pour optimiser vos images Docker.

## 1. Optez pour des images de base légères

Commencer avec une image légère, c’est comme construire une maison avec des briques plutôt que des rochers. Faites le bon choix dès le départ.

### Exemple Spring Boot :

```dockerfile
# Mauvaise idée
FROM eclipse-temurin:21

# Bonne pratique
FROM eclipse-temurin:21-jre-alpine
```

L’image `eclipse-temurin:21-jre-alpine` offre tout ce dont vous avez besoin sans les extras inutiles. Moins de poids, plus de vitesse.

---

## 2. Faites le ménage

Installer des dépendances, c’est bien, mais pensez à faire le ménage après pour ne pas encombrer votre image avec des fichiers temporaires inutiles.

### Exemple :

```dockerfile
RUN apt-get update 
RUN apt-get install -y openjdk-21-jre 
RUN apt-get clean 
RUN rm -rf /var/lib/apt/lists/*
```

Le ménage post-installation garantit une image propre et efficace. Pas de déchets !

Chaque commande RUN effectue une tâche spécifique. Cela peut aider à déboguer, mais attention : cela ajoute des couches inutiles. Voyons pourquoi dans la section suivante.

---

## 3. Utilisez .dockerignore

Comme avec `.gitignore` , le fichier `.dockerignore` vous permet d’exclure tout ce qui n’a pas besoin d’être dans votre image Docker (et croyez-moi, il y en a souvent beaucoup).

### Exemple de `.dockerignore` :

```
target/
.git
```

En ignorant des dossiers comme `target/` ou `.git` , vous réduisez directement la taille de votre image. Gardez uniquement l’essentiel.

---

## 4. Minimisez les couches (Un Dockerfile épuré, c’est toujours mieux)

Chaque instruction `RUN` , `COPY` ou `ADD` ajoute une couche à votre image. Trop de couches ? Trop de poids ! Combinez-les pour un Dockerfile plus compact.

### Exemple :

```dockerfile
RUN apt-get update && apt-get install -y \  
    openjdk-21-jre && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

⚠️ Contrairement à l’exemple précédent dans la section "Faites le ménage", ici les commandes RUN sont combinées pour minimiser le nombre de couches. C’est clairement la meilleure pratique pour une image Docker optimisée.

Moins de couches, c’est une image plus légère et une construction plus rapide. Simple, non ?

---

## 5. Builds multi-steps : Faites simple, faites efficace

Les builds multi-steps permettent de tout faire dans une première étape (compilation, tests, etc.) et de ne garder que le strict nécessaire dans l’image finale.

### Exemple avec une application Spring Boot :

```dockerfile
# Étape 1 : Compilation
FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Étape 2 : Déploiement
FROM eclipse-temurin:21-jre-alpine
COPY --from=build /app/target/*.jar /app/app.jar
CMD ["java", "-jar", "/app/app.jar"]
```

Ce processus divise le travail en étapes claires et optimise le résultat final. Votre image sera légère et prête pour la production.

---

## Conclusion

En appliquant ces astuces, vous pouvez transformer vos images Docker en véritables modèles d’efficacité. Moins de poids, plus de rapidité, et des déploiements fluides 🚀.
