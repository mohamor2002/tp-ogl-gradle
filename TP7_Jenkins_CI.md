# TP N°7 – Intégration continue d’une API Java avec Jenkins

## École nationale Supérieure d’Informatique  
**2CS SIL – Outils de Génie Logiciel**

---

## Objectif
Il s’agit de développer le processus d’intégration continue d’une API Java.  
Le pipeline CI est structuré en plusieurs phases décrites ci-dessous.

---

## 1. Création du projet

1. Créer un projet **Gradle** et intégrer à la racine du projet :
   - le dossier `src`
   - le fichier `build.gradle`
   - le dossier `feature`
   - le fichier `Jenkinsfile`  

   ⚠️ Conserver dans `build.gradle` les plugins :
   - Jacoco  
   - Sonar  
   - maven-publish  

2. Partager le code source sur **GitHub**.

3. Après le lancement de Jenkins et l’installation des plugins nécessaires, créer un **Multibranch Pipeline Project**.

4. Ajouter le **webhook** de l’instance Jenkins dans le repository GitHub.

---

## 2. Les phases du pipeline

### 2.1 Phase Test
Cette phase comprend :
1. Lancement des tests unitaires  
2. Archivage des résultats des tests unitaires  
3. Génération des rapports de tests **Cucumber**

---

### 2.2 Phase Code Analysis
Analyse de la qualité du code à l’aide de **SonarQube**.

---

### 2.3 Phase Code Quality
Vérification de l’état des **Quality Gates** :
- Si l’état est **Failed**, l’exécution du pipeline doit s’arrêter.

---

### 2.4 Phase Build
Cette phase comprend :
1. Génération du fichier **JAR**
2. Génération de la documentation
3. Archivage du fichier JAR et de la documentation

---

### 2.5 Phase Deploy
Déploiement du fichier JAR généré sur :  
👉 https://mymavenrepo.com/

---

### 2.6 Phase Notification
Envoi de notifications à l’équipe de développement :
- En cas de succès : notification par **mail** et sur **Slack**
- En cas d’échec dans l’une des phases : notification d’erreur à l’équipe

---
