# Suivi des TPs du jour 4  

## Installation Jenkins avec les plugins nécessaires


### A/ Démarrage

docker network create jenkins 


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


## b/  Construction d'un Container Jenkins

`mkdir jenkins && cd jenkins `

`nano Dockerfile `


Y coller le contenu suivant : 

```
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

`docker build -t myjenkins-blueocean:2.414.1-1 .`




## B/ Installer les Plugins par défaut et configurations initiales 
 
  **Connexion**: `http://localhost:8080`
  
  Entrer le mot de passe obtenu depuis les logs du container
  
  
  **URL jenkins**: `http://<ip_jenkins>:8080`
  
  **Connaître l'ip jenkins** : 

	docker inspect jenkins-blueocean | jq '.[0].NetworkSettings'
	
  Récupérer la valeur du Champ : IpAddress
  
  **Email**: Utiliser l'adresse email que vous utiliserez comme passerelle pour le serveur de mail.

  
## C/ Exploration du répertoire jenkins_home
 
  `sudo ls -l /var/lib/docker/volumes/jenkins-data/_data`
  
  il contient les dossiers que nous avons listés comme étant le coeur de l'installation Jenkins 
 
 
 ## E/ Exécution notre premier Job sous Jenkins 


 - Créer le job pour compiler et empaquetter toute nouveau code.
   - il va détecter automatiquement les nouvelles modifications et les nouvelles branches 
   
Nous aurons besoin d'un multibranch pipeline : 

   nom: zerofiltre_multibranch
   display name : Zerofiltre Multibranch
   
   Source: Git
   
   Ajouter les identifiants :  add => Jenkins 
   Car nous souhaitons que ces identifiants soient utilisés par toute l'install jenkins et pas uniquement pour ce pipeline 
   
   Laissez les params de création de credentials par défaut: entrez votre user + pwd github.
   
   Attention aux id de credentials, ils peuvent être réutilisés dans les scripts de pipeline par la suite (évitez les espaces et 
   les caractères spéciaux)
   
   
   Repository https url : l'url github de votre clone 
   
   
   Stratégie: Laisser toutes les stratégies par défaut 
   
   
   Scan multibranch Pipeline Triggers: 
	Check périodique de 5 mins (le check pourra aussi se faire manuellement)
	
	
   LEs autres params par défaut .
   
   
   Il va faire un checkout des différentes branches mais échouer car aucun fichier Jenkinsfile n'est présent.
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
  E/ Ecriture du premier script Jenkinsfile 

   
		Ouvrir le projet et se déplacer sur la branche : email-on_article_in_review_inow_jour4
		
		Avant de commencer à écrire , un peu de théorie sur les pipelines
		
	
	
		
	Notre script : 
	
	Avant d'exécuter ce script, il ca falloir installer ces tools là sous jenkins 
	
	Dashboard => Manage jenkins => Tools  
	
		JDK Installations => Add jdk  ==> donner le nom exact utilisé dans le pipeline dans la partie tools : Laisser install Automatically
	 
		Maven Installation => Add maven ==> donner le nom exact utilisé dans le pipeline dans la partie tools (Maven3) : Laisser install Automatically
	
   
   
   
   
   F/ Ajouter une phase de test + rapport de test après build 
   
   
   Dans le projet, créer le fichier Jenkins et ajoutez y ceci :
   
   
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
		
	
	
		Si les tests échouent avec la raison :   Corrupted STDOUT by directly writing to native stream in forked JVM 1. See FAQ web page and the dump file 
		
		Il s'agit d'un bug de la version de jacoco, faire passer la version du plugin à 0.8.4 and jococo plugin to 0.8.8
		
		
		
		
		
		
		
		Référentiel d'utilitaires à exploiter dans les steps :
		https://www.jenkins.io/doc/pipeline/steps/
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	G/ Ajouter un rapport de couverture de test :https://plugins.jenkins.io/jacoco/
	

	


	
		
		
		
		
		
	
		

    H/ Intégration à Sonarqube 
	
	
	i/ Sous jenkins : 
	
	Intaller le plugin : SonarQube scanner 
	
	
	
	Aller sous sonarqube générer un token pour jenkins :
	
	Gébérer un token pour jenkins :  squ_a8969e37a24723e3ebcc61979ea777d00889bfbd
	
	Puis Manage Jenkins : 
		=> System : Add sonarqube server 
		
	    Name: SonarqubeServer 
		
		Connecter le container sonar au sous-réseau jenkins: docker network connect jenkins sonarqube
		
		docker inspect sonarqube | jq '.[0].NetworkSettings'
		
		@IP Server : http://<ipsonar_reseau_jenkins>:9000
		
		
		Secret : Add => secret de type scret key  avec le toke créé plus haut 
			
		Ajouter le secret et sauvegarder


	
	ii/ Sous sonar Configurer un webhook de retour vers Jenkins 
	
	Administration > Configuration > Webhooks
	
	URL du webhook: http://<ip_jenkins>:8090/sonarqube-webhook/
	
	
	
	
	Référentiel sur les webhooks sonar:
	http://localhost:9000/documentation/project-administration/webhooks/
	
	
	Référentiel d'intégration à sonarqube: 
	https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/jenkins-extension-sonarqube/
	
	
	
	
	
	iii/ Ajouter un stage analyse sonar dans le build 
	
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
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	J/ Déploiement sur artifactory 
	
	i/ Définition des vraibles d'environnement système à Jenkins 
	En effet nous voulons exécuter mvn deploy -s .m2/settins.xml qui dépend des 
	variables d'env de connexion à artifactory 
	
	Nous les avons défini dans ~/.bashrc 
	
	ARTIFACTORY_SERVER_USERNAME=...
	ARTIFACTORY_SERVER_PASSWORD=...
	
	Variables d'environnement globales Jenkins: 
		Injectées dans tous les pipeline de tout agent sous la forme :
		${NOM_VARIABLE} dans un projet freestyle sans jenkinsfile
		
		Dans un pipeline: env.NOM_VARIABLE
		
	
	ii/ Intégration script groovy dans un pipeline déclaratif via la directive script. 
	
	    stage('Package and Deploy') {
            steps {
                script {
                    // Check if the branch name starts with 'release-'
                    if (env.BRANCH_NAME.startsWith('release-')) {
                        // Extract the version from the branch name
                        String version = env.BRANCH_NAME.replaceAll(/^release-/, '')

                        // Set the project version using Maven
                        sh "mvn versions:set -DnewVersion=${version}"
                    }

                    // Deploy the project (always)
                    sh 'mvn deploy -DskipTests .m2/settings.xml'

                    // Archive the deployed artifacts
                    archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                }
            }
        }
	
	
	









	
	
	H/ Notification par mail 
	
	
	Configuration du client de mail :
	
		Installer le plugin Email extension Plugin

		Puis : 

			Manage jenkins ==> Manage system ==> défiler jusqu'à Extended E-mail Notification et non Email Notification

			Configuration:

			Serveur SMTP: smtp.gmail.com
			Port: 587

			Use TLS :

            Add credentials :			
				username: Gmail email address , 
				password : Google App password
				
				How to generate the email app password : 
					https://support.google.com/accounts/answer/185833?hl=en

			+

			Use TLS
			
			
		
	
	
		
   
   
  



NOTIONS DE SECURITE JENKINS 

	i/ création d’un nouveau user et connexion de ce dernier
	
	MAnage Jenkins => Users => Create User 
	
	


    ii/ Mise en place d’une gestion granulaires des rôles

	Dashboard => Manage Jenkins => Role-based authorization Strategy
	
	Puis Dashboard => Manage Jenkins => Security => Authrozation => Role Based Authorization Strategy : save 
	
	Si vous ne voyez pas "authorization", redémarrez Jenkins 
	
	Dashboard => Manage Jenkins => Manage and Assign roles 

	
	Ajouter un Role pour les utilisateurs de jobs :
	Ils pourront : créer, démarrer, configurer, déplacer, ... mais il ne pourront 
	pas les supprimer.
	
	Mais ils ne pourront pas administrer 
	
	Rassurez-vous que vous n'êtes pas connecté avec l'utilisateur créé précédemment, sinon déconnectez-vous et connectez
	vous avec un autre user, car nous allons restreindre les permission de l'utilisateur précédent.
	
	Dans la case "Role to add" : JobUser 
	
	Et cocher :

	dans le cadran Overall les cases : Read
	dans le cadran Jobs les cases: Workspace, Read, Move, create, configure, cancel,build 
	
	puis save.
	
	Dans la case assign roles : 
		Ajouter le user précédent à JobUser 
		et le user actuel à admin
	
	
	Save puis se connecter avec le user précédent pour voir à quoi il a accès.
	
	Pas possible de supprimer un job
	
	
	
			Si vous vous vérrouillez à l'extérieur:

			Retrouver le fichier config.xml dans JENKINS_HOME


			<hudson>
				...
				<numExecutors>2</numExecutors>
				<mode>NORMAL</mode>
				<useSecurity>true</useSecurity>
				...

			Changez la valeur de <useSecurity> à false, et redémarrez votre serveur. 
			
			Vous pourrez alors à nouveau accéder à Jenkins, et mettre en place votre configuration de sécurité correctement. 
	
	
	
	
	
	
	
	
	
	
	iii/ Suivi des configurations 
	
	Installer le plugin Job configuration History 
	
	Puis : Dashboard => Job config history pour le listing
	
	
	
	Manage jenkins : => Job Config History  pour la config, fichier à exclure, emplacement de l'historique, etc.

 
 
 
 
 
