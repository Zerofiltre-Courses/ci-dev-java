# Suivi des TPs du jour 4

## Installation de Jenkins avec les plugins nécessaires

### A/ Démarrage

```sh
docker network create jenkins
```

## a/ Container dind : Car nous allons utiliser des agents docker

```sh
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 3000:3000 --publish 5000:5000 --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```

## b/ Construction d'un Container Jenkins

1. Créez un répertoire Jenkins :

```sh
mkdir jenkins && cd jenkins
```

2. Créez un fichier Dockerfile et collez-y le contenu suivant :

```Dockerfile
FROM jenkins/jenkins:2.414.1-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.27.6 docker-workflow:572.v950f58993843"
```

3. Construisez l'image Docker :

```sh
docker build -t myjenkins-blueocean:2.414.1-1 .
```

## B/ Installer les Plugins par défaut et configurations initiales

- **Connexion**: `http://localhost:8080`
- Entrez le mot de passe obtenu depuis les logs du container.

- **URL Jenkins**: `http://<ip_jenkins>:8080`
- Pour connaître l'adresse IP de Jenkins :

```sh
docker inspect jenkins-blueocean | jq '.[0].NetworkSettings.IPAddress'
```

- **Email**: Utilisez l'adresse e-mail que vous utiliserez comme passerelle pour le serveur de messagerie.

## C/ Exploration du répertoire jenkins_home

```sh
sudo ls -l /var/lib/docker/volumes/jenkins-data/_data
```

Ce répertoire contient les dossiers que nous avons listés comme étant le cœur de l'installation Jenkins.

## D/ Exécution de notre premier Job sous Jenkins

- Créez le job pour compiler et empaqueter tout nouveau code.
- Il détectera automatiquement les nouvelles modifications et les nouvelles branches.

Nous aurons besoin d'un multibranch pipeline :

- Nom : `zerofiltre_multibranch`
- Display name : `Zerofiltre Multibranch`
- Source : Git
- Ajoutez les identifiants : ajoutez => Jenkins, car nous souhaitons que ces identifiants soient utilisés par toute l'installation Jenkins, et pas uniquement pour ce pipeline.
- Laissez les paramètres de création de credentials par défaut : entrez votre nom d'utilisateur GitHub et mot de passe.
- Attention aux ID de credentials, ils peuvent être réutilisés dans les scripts de pipeline par la suite (évitez les espaces et les caractères spéciaux).
- Repository https URL : l'URL GitHub de votre clone
- Stratégie : Laissez toutes les stratégies par défaut
- Scan multibranch Pipeline Triggers : Vérifiez périodiquement toutes les 5 minutes (le check peut également se faire manuellement).
- Les autres paramètres restent par défaut.

Le pipeline fera un checkout des différentes branches mais échouera car aucun fichier Jenkinsfile n'est présent.

## E/ Écriture du premier script Jenkinsfile

1. Ouvrez le projet et accédez à la branche : `email-on_article_in_review_inow_jour4`.

2. Avant de commencer à écrire le script, installez ces outils sous Jenkins :

   - Dashboard => Manage Jenkins => Tools
     - JDK Installations => Add JDK
       - Donnez le nom exact utilisé dans le pipeline dans la section "Tools"
       - Laissez "Install Automatically" activé
     - Maven Installation => Add Maven
       - Donnez le nom exact utilisé dans le pipeline dans la section "Tools" (Maven3)
       - Laissez "Install Automatically" activé

## F/ Ajout d'une phase de test + rapport de test après la construction

Dans le projet, créez le fichier Jenkins et ajoutez-y ceci :

```groovy
stage('Test') {
    steps {
        sh 'mvn test'
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
```

Si les tests échouent avec la raison : "Corrupted STDOUT by directly writing to native stream in forked JVM 1. See FAQ web page and the dump file", il s'agit d'un bug de la version de Jacoco. Mettez à jour la version du plugin à 0.8.4 et le plugin Jacoco à 0.8.8.

Référentiel d'utilitaires à exploiter dans les étapes : [Jenkins Pipeline Steps](https://www.jenkins.io/doc/pipeline/steps/)

## G/ Ajout d'un rapport de couverture de test

Utilisez le plugin [Jacoco](https://plugins.jenkins.io/jacoco/).

## H/ Intégration avec SonarQube

### i/ Sous Jenkins

1. Installez le plugin : SonarQube Scanner.

2. Allez sous SonarQube pour générer un token pour Jenkins :
   - Générez un token pour Jenkins : `squ_a8969e37a24723e3ebcc61979ea777d00889bfbd`

3. Puis, dans Manage Jenkins :
   - System => Ajoutez un serveur SonarQube
     - Nom : `SonarqubeServer`
     - Connectez le conteneur Sonar au sous-réseau Jenkins : `docker network connect jenkins sonarqube`
     - Obtenez l'adresse IP du serveur : `docker inspect sonarqube | jq '.[0].NetworkSettings.IPAddress'`
     - URL du serveur SonarQube : `http://<ipsonar_reseau_jenkins>:9000`
     - Ajoutez le secret de type "Secret Key" avec le token créé préc

édemment.

### ii/ Sous SonarQube

1. Configurez un webhook de retour vers Jenkins :
   - Administration => Configuration => Webhooks
   - URL du webhook : `http://<ip_jenkins>:8090/sonarqube-webhook/`
   - Référentiel sur les webhooks Sonar : [Webhooks Sonar](http://localhost:9000/documentation/project-administration/webhooks/)
   - Référentiel d'intégration à SonarQube : [Intégration SonarQube](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/)

### iii/ Ajout d'un stage d'analyse Sonar dans la construction

```groovy
stage('SonarQube analysis') {
    steps {
        withSonarQubeEnv('SonarQubeServer') {
            sh 'mvn sonar:sonar -s .m2/settings.xml'
        }
    }
}
stage('Quality Gate') {
    steps {
        timeout(time: 2, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

## J/ Déploiement sur Artifactory

### i/ Définition des variables d'environnement système à Jenkins

Nous voulons exécuter `mvn deploy -s .m2/settings.xml`, qui dépend des variables d'environnement de connexion à Artifactory. Nous les avons définis dans `~/.bashrc`.

- `ARTIFACTORY_SERVER_USERNAME=...`
- `ARTIFACTORY_SERVER_PASSWORD=...`

Variables d'environnement globales Jenkins :
- Injectées dans tous les pipelines de tout agent sous la forme : `${NOM_VARIABLE}` dans un projet freestyle sans Jenkinsfile.
- Dans un pipeline : `env.NOM_VARIABLE`

### ii/ Intégration de script Groovy dans un pipeline déclaratif via la directive script

```groovy
stage('Package and Deploy') {
    steps {
        script {
            // Vérifiez si le nom de la branche commence par 'release-'
            if (env.BRANCH_NAME.startsWith('release-')) {
                // Extrait la version du nom de la branche
                String version = env.BRANCH_NAME.replaceAll(/^release-/, '')

                // Définit la version du projet en utilisant Maven
                sh "mvn versions:set -DnewVersion=${version}"
            }

            // Déployez le projet (toujours)
            sh 'mvn deploy -DskipTests .m2/settings.xml'

            // Archivez les artefacts déployés
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
    }
}
```

## H/ Notification par e-mail

### Configuration du client de messagerie

1. Installez le plugin Email Extension Plugin.

2. Puis, dans Manage Jenkins => Manage system, faites défiler jusqu'à "Extended Email Notification" (et non "Email Notification").

3. Configuration :
   - Serveur SMTP : `smtp.gmail.com`
   - Port : `587`
   - Utilisez TLS.
   - Ajoutez les informations d'identification :
     - Nom d'utilisateur : adresse e-mail Gmail
     - Mot de passe : Mot de passe de l'application Google (comment générer le mot de passe de l'application : [Lien](https://support.google.com/accounts/answer/185833?hl=en))
   - Utilisez TLS.

## NOTIONS DE SÉCURITÉ JENKINS

### i/ Création d'un nouveau utilisateur et connexion de ce dernier

- Manage Jenkins => Users => Create User

### ii/ Mise en place d'une gestion granulaire des rôles

- Dashboard => Manage Jenkins => Role-based Authorization Strategy

Puis, Dashboard => Manage Jenkins => Security => Authorization => Role Based Authorization Strategy : save. Si vous ne voyez pas "authorization", redémarrez Jenkins.

Dashboard => Manage Jenkins => Manage and Assign Roles

Ajoutez un rôle pour les utilisateurs de jobs :
- Ils pourront créer, démarrer, configurer, déplacer, mais ne pourront pas supprimer les jobs.
- Ils ne pourront pas administrer.

Assurez-vous que vous n'êtes pas connecté avec l'utilisateur créé précédemment, sinon déconnectez-vous et connectez-vous avec un autre utilisateur, car nous allons restreindre les permissions de l'utilisateur précédent.

Dans la case "Role to add" : `JobUser`
Cochez :
- Dans le cadran Overall : les cases Read
- Dans le cadran Jobs : les cases Workspace, Read, Move, Create, Configure, Cancel, Build

Ensuite, ajoutez l'utilisateur précédent à JobUser et l'utilisateur actuel à Admin dans la case "Assign Roles".

Sauvegardez, puis connectez-vous avec l'utilisateur précédent pour voir à quoi il a accès. Il ne sera pas possible de supprimer un job.

Si vous vous verrouillez à l'extérieur, retrouvez le fichier `config.xml` dans JENKINS_HOME, changez la valeur de `<useSecurity>` à `false`, puis redémarrez votre serveur. Vous pourrez alors à nouveau accéder à Jenkins et mettre en place votre configuration de sécurité correctement.

### iii/ Suivi des configurations

Installez le plugin "Job Configuration History" pour suivre les modifications de configuration :
- Dashboard => Manage Jenkins => Manage Plugins => Available (recherchez "Job Configuration History")

Puis, accédez à : Dashboard => Job Config History pour la liste des modifications.

Dans Manage Jenkins => Job Config History, configurez les fichiers à exclure, l'emplacement de l'historique, etc.