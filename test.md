# 🧪 TP DevOps Java – Mise en place d’un pipeline CI complet

## 🎯 Objectif pédagogique

Dans ce TP, vous allez reproduire un **pipeline CI DevOps complet**, similaire à celui utilisé en environnement professionnel.

Vous allez comprendre et mettre en place :

* Le build d’une application Java avec Maven
* L’exécution de tests automatisés
* L’analyse de qualité du code avec SonarQube
* La publication d’un artefact dans Nexus
* L’automatisation avec GitHub Actions

👉 Important : vous devez **reproduire exactement les étapes**, pas improviser.

---

# 🧱 1. Structure du projet

## 🔎 Pourquoi cette étape ?

Un projet Maven repose sur une **structure standardisée**.
Cette structure permet à Maven de savoir :

* où se trouve le code
* où se trouvent les tests
* comment exécuter le projet

---

## 📁 Arborescence à créer

```text id="tree_final"
text-utils/
│── pom.xml
│
└── src/
    ├── main/
    │   └── java/
    │       └── com/
    │           └── devops/
    │               └── TextUtils.java
    │
    └── test/
        └── java/
            └── com/
                └── devops/
                    └── TextUtilsTest.java
```

👉 Travail demandé :
Reproduisez exactement cette structure.

---

# 💻 2. Implémentation de l’application

## 🔎 Pourquoi cette application ?

Elle permet de :

* manipuler des données (String)
* tester plusieurs cas (logique + transformation)
* écrire facilement des tests unitaires

---

## ➤ Fichier : `TextUtils.java`

```java id="code_main"
package com.devops;

public class TextUtils {

    public String reverse(String input) {
        return new StringBuilder(input).reverse().toString();
    }

    public boolean isPalindrome(String input) {
        String reversed = reverse(input);
        return input.equalsIgnoreCase(reversed);
    }

    public int countWords(String input) {
        if (input == null || input.trim().isEmpty()) return 0;
        return input.trim().split("\\s+").length;
    }

    public String toUpperCase(String input) {
        return input.toUpperCase();
    }
}
```

---

## ➤ Fichier : `TextUtilsTest.java`

## 🔎 Pourquoi les tests ?

Ils permettent de :

* vérifier automatiquement le bon fonctionnement
* éviter les régressions
* valider le build

```java id="code_test"
package com.devops;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class TextUtilsTest {

    TextUtils utils = new TextUtils();

    @Test
    void testReverse() {
        assertEquals("cba", utils.reverse("abc"));
    }

    @Test
    void testPalindrome() {
        assertTrue(utils.isPalindrome("radar"));
    }

    @Test
    void testWordCount() {
        assertEquals(3, utils.countWords("Java is cool"));
    }

    @Test
    void testUpperCase() {
        assertEquals("HELLO", utils.toUpperCase("hello"));
    }
}
```

---

# ⚙️ 3. Configuration Maven

## 🔎 Pourquoi Maven ?

Maven automatise :

* la compilation
* les dépendances
* l’exécution des tests

---

## ➤ Fichier `pom.xml` (version initiale)

```xml id="pom_init"
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.devops</groupId>
  <artifactId>text-utils</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.1.2</version>
      </plugin>
    </plugins>
  </build>
</project>
```

---

# ▶️ 4. Exécution des tests

```bash id="run_cmd"
mvn clean verify
```

## 🔎 Ce que fait cette commande

* Compile le projet
* Exécute les tests
* Vérifie que tout est valide

👉 Si cette étape échoue, **la suite du pipeline échouera aussi**

---

# 🔍 5. Analyse de code avec SonarQube

## 🔎 Pourquoi SonarQube ?

SonarQube permet de :

* détecter des bugs
* identifier les mauvaises pratiques
* mesurer la qualité du code

---

## Étape 1 : Générer un token

* Connectez-vous à SonarQube
* My Account → Security
* Générez un token

---

## Étape 2 : Modifier le `pom.xml`

Ajoutez dans `<properties>` :

```xml id="sonar_props"
<sonar.projectKey>java-sonar-project</sonar.projectKey>
<sonar.host.url>http://<VOTRE_IP>:9000</sonar.host.url>
```

---

Ajoutez le plugin :

```xml id="sonar_plugin"
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.10.0.2594</version>
</plugin>
```

---

## Étape 3 : Lancer l’analyse

```bash id="sonar_cmd"
mvn sonar:sonar -Dsonar.login=VOTRE_TOKEN
```

---

# 📦 6. Publication avec Nexus

## 🔎 Pourquoi Nexus ?

Nexus sert à :

* stocker les artefacts (JAR)
* centraliser les versions
* permettre leur réutilisation

---

## Étape 1 : Créer un utilisateur Nexus

Créer un utilisateur avec droits de déploiement.

---

## Étape 2 : Modifier le `pom.xml`

Ajoutez :

```xml id="nexus_block"
<distributionManagement>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <url>http://<VOTRE_IP>:8081/repository/maven-snapshots/</url>
  </snapshotRepository>
  <repository>
    <id>nexus-releases</id>
    <url>http://<VOTRE_IP>:8081/repository/maven-releases/</url>
  </repository>
</distributionManagement>
```

---

## Étape 3 : Configurer `settings.xml`

```xml id="settings_xml"
<settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>USER</username>
      <password>PASSWORD</password>
    </server>
    <server>
      <id>nexus-releases</id>
      <username>USER</username>
      <password>PASSWORD</password>
    </server>
  </servers>
</settings>
```

---

## Étape 4 : Déployer

```bash id="deploy_cmd"
mvn deploy
```

---

# 🧾 7. Création du dépôt GitHub

## 🔎 Pourquoi ?

Git permet de versionner le code et déclencher le pipeline CI.

---

## Étapes

1. Aller sur GitHub
2. Cliquer sur **New repository**
3. Nom : `text-utils`
4. Ne rien cocher

---

## Initialisation locale

```bash id="git_init"
git init
git branch -m main
git add .
git commit -m "Initial commit"
git remote add origin <URL_DU_REPO>
git push -u origin main
```

---

# 🌿 8. Branche develop

```bash id="git_dev"
git checkout -b develop
git push -u origin develop
```

👉 Organisation :

* main → production
* develop → développement

---

# ⚙️ 9. Pipeline CI complet

## 🔎 Pourquoi ?

Automatiser toutes les étapes :

* build
* test
* analyse
* déploiement

---

## Créer :

```id="workflow_path"
.github/workflows/ci.yml
```

---

## ➤ Pipeline (référence Igor)

```yaml id="pipeline_full"
name: CI Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-test-analyze-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0 

    - name: Setup JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'

    - name: Build & Tests
      run: mvn clean verify

    - name: SonarQube Scan
      run: |
        mvn sonar:sonar \
          -Dsonar.projectKey=java-sonar-project \
          -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
          -Dsonar.login=${{ secrets.SONAR_TOKEN }}

    - name: Configure Maven settings.xml
      run: |
        mkdir -p ~/.m2
        cat << EOF > ~/.m2/settings.xml
        <settings>
          <servers>
            <server>
              <id>nexus-snapshots</id>
              <username>${{ secrets.NEXUS_USERNAME }}</username>
              <password>${{ secrets.NEXUS_PASSWORD }}</password>
            </server>
            <server>
              <id>nexus-releases</id>
              <username>${{ secrets.NEXUS_USERNAME }}</username>
              <password>${{ secrets.NEXUS_PASSWORD }}</password>
            </server>
          </servers>
        </settings>
        EOF

    - name: Deploy to Nexus
      run: mvn deploy -DskipTests
```

---

# 🔐 10. Configuration des secrets GitHub

Dans votre repository :

Settings → Secrets → Actions

Ajouter :

* SONAR_HOST_URL
* SONAR_TOKEN
* NEXUS_USERNAME
* NEXUS_PASSWORD

---

# 🏷️ 11. Badge du pipeline

Ajoutez dans votre README :

```md id="badge_final"
![CI](https://github.com/<USER>/<REPO>/actions/workflows/ci.yml/badge.svg)
```

---

# ✅ Résultat attendu

À la fin :

* ✔️ Projet Maven fonctionnel
* ✔️ Tests automatisés
* ✔️ Analyse SonarQube OK
* ✔️ Artefact publié dans Nexus
* ✔️ Pipeline CI identique à celui de Igor

---

# 📌 Important

Vous devez être capable d’expliquer :

* le rôle de chaque étape
* pourquoi on automatise
* ce qui se passe si une étape échoue

---

# 🚀 Fin du TP
