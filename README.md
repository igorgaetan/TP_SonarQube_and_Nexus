
# 🚀 Projet : Pipeline de Qualité Continue (Java/SonarQube)
**Support de Cours & Travaux Pratiques — Ingénierie DevOps**

Ce projet simule un flux de travail industriel. L'objectif est d'implémenter une barrière de qualité (Quality Gate) automatisée pour une application Java en utilisant **Maven** et **Gradle**.

---

## 🎯 Objectifs Pédagogiques
1. **Industrialiser** la compilation et la gestion des dépendances.
2. **Automatiser** les tests unitaires et la mesure de couverture (**JaCoCo**).
3. **Piloter** la conformité logicielle via **SonarQube**.
4. **Déployer** une stack technique complète sur **AWS EC2** via **Docker**.

---

## 🛠 Stack Technique
* **Langage :** Java 21 (LTS)
* **Build Automation :** Maven 3.x & Gradle 8.7
* **Qualité :** SonarQube LTS (Community) & JaCoCo
* **Infrastructure :** Docker sur Ubuntu (AWS EC2)

---

## 📂 Architecture du Projet
Le dépôt est configuré pour être "Poly-Build" (compatible Maven et Gradle simultanément).

```bash
java-calculator-devops-lab/
├── src/main/java/.../Calculator.java      # Code source
├── src/test/java/.../CalculatorTest.java  # Tests JUnit 5
├── pom.xml                                # Script Maven
├── build.gradle                           # Script Gradle
└── sonar-project.properties               # Configuration Scanner
```

---

## ⚙️ Guide d'Installation (Provisioning)

### 1. Préparation d'Ubuntu & Java 21
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl unzip openjdk-21-jdk -y
```

### 2. Configuration de Gradle 8.7
*Indispensable pour la compatibilité avec Java 21.*
```bash
wget https://services.gradle.org/distributions/gradle-8.7-bin.zip
unzip gradle-8.7-bin.zip
sudo mv gradle-8.7 /opt/gradle
echo 'export PATH=$PATH:/opt/gradle/bin' >> ~/.bashrc && source ~/.bashrc
```

### 3. Déploiement de SonarQube via Docker
```bash
sudo apt install docker.io -y
sudo usermod -aG docker $USER  # Nécessite une reconnexion
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

---

## 🔍 Exécution de l'Analyse

### Étape 1 : Récupération du Lab
```bash
git clone https://github.com/igorgaetan/java-calculator-devops-lab.git
cd java-calculator-devops-lab
```

### Étape 2 : Lancement du Scan




**Option A : Maven**
```bash
mvn clean verify sonar:sonar \
  -Dsonar.login=admin \
  -Dsonar.password=VOTRE_PASSWORD
```

**Option B : Gradle**
```bash
gradle clean build jacocoTestReport sonar \
  -Dsonar.login=TON_TOKEN_GENERE
```

---

## ✅ Le Verdict : Quality Gate

Pour autoriser le stockage de l'artéfact final (ex: dans **Nexus**), le projet doit impérativement valider les critères suivants :

| Métrique | Seuil Critique |
| :--- | :--- |
| **Tests Unitaires** | **100% Pass** (5/5) |
| **Couverture (JaCoCo)** | **$\ge$ 80%** |
| **Bugs & Vulnérabilités** | **0** (Bloquant) |
| **Dette Technique** | **< 5%** |



---

> **💡 Note :** Dans un cycle CI/CD réel, si SonarQube renvoie un statut **FAILED**, le pipeline s'arrête immédiatement. On n'envoie jamais un code "sale" en production.