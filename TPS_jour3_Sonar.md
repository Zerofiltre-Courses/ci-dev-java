# Suivi des TPS du jour 3

## A/ Installation sonarqube 'on premise'

Installation avec une BD h2 en mémoire :

* h2 => prendre la main (données non persistées)

* postgresql ou mysql pour une utilisation en prod 


`docker run --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest`


Ouvrir l'interface graphique :
`http://localhost:9000`


identifiants par défaut: 
username/password : `admin/admin`


## B/ Collecter des métriques et les publier via l'agent sonarScanner pour maven 


Nous allon utiliser le scaner sonar pour maven:

https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/



Se déplacer sur la branche  :

`git co email-on_article_in_review_inow_jour3`


Dans le fichier .m2/settings.xml  ajouter les sections :

    

   
````xml
<profiles>
      ...
  <profile>
      <id>sonar</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <sonar.host.url>
          http://localhost:9000
        </sonar.host.url>
        <sonar.token>
          ${env.SONAR_TOKEN}
        </sonar.token>
      </properties>
  </profile>
  
  ...
 
</profiles>
````



## C/ Créer un token pour faire authentifier l'application auprès de sonar 

`https://localhost:9000`

**Administration > Security > Users > Token**

Cliquer sur le menu Hamburger pour générer un token.
Donnez-lui un nom: `Zerofiltre Token`

Générez, copiez et sauvegardez!



Exportez cette valeur dans la variable d'environnement SONAR_TOKEN

````
echo "export SONAR_TOKEN=votre_token_administrateur" >> ~/.bashrc

source ~/.bashrc
````

Puis :

```
mvn clean package sonar:sonar -s .m2/settings.xml
```

En cas de non atteinte de serveur sonar :

Vérifiez que le container est bel et bien démarré.
Sinon démarrez le : 

`docker ps -a`

Vérifiez qu'il est en cours d'exécution, s'il est arrêté démarrez le :
	
`docker start sonarqube`

ouvrir les logs:
`docker logs -f --tail=100 sonarqube`

et attendez que cette ligne de log s'affiche:

`INFO  app[][o.s.a.SchedulerImpl] SonarQube is operational`



Puis relancer:

`mvn clean package sonar:sonar -s .m2/settings.xml`

Une fois l'analyse faite: dirigez-vous vers : 
````
http://localhost:9000/dashboard?id=tech.zerofiltre%3Ablog

````
pour observer les résultats.





## D/ Les quality gate 

Nous allons nous intérésser aux quality gates 

Nous nous rendons compte que les seuils de métrique ne correspondent pas au résultat obtenu: ==> pas de nouveau code considéré.

Il nous faut ajouter du code, refaire une analyse et soumettre de nouveau.





### a/ Taux de couverture 

Commençons par le taux de couverture: Beaucoup trop de classes marquées commes non couvertes que nous n'avons pas besoin de tester: 

Ajoutons cette partie au fichier settings.xml :

```xml

 <sonar.coverage.exclusions>
                    **/article/model/**/*,**/logging/model/**/*,**/user/model/**/*,**/mapper/**/*,**/payment/model/**/*,**/*JPA.*,**/Page.*,**/SpringPageMapper.*,
                    **/config/*,**/error/*,**/filter/*,**/*EntryPoint, **/ZerofiltreBlogApplication.*, **/util/*,
                    **/*Exception.*,**/logging/**/*,**/InfraProperties.*,**/*VM.*,**/entrypoints/rest/**/*,**/course/model/**/*,**/ovh/model/**/*
  </sonar.coverage.exclusions>

```


`mvn clean package sonar:sonar -s .m2/settings.xml`


### b/ Bugs

Corrigez les bugs 


### c/ Security hotspots 

Comprendre les implications et Bypasser 


### d/ Blocs de code dupliqués

Analysez et factorisez

Refaire une analyse et observez les résultats



## E/ Définition de quality Gate personnalisés

Vous avez la possibilité de personnaliser les quality gate :


Quality gate => create => editer les params 




## E/ Le plugin sonar lint pour IDE : Intellij , vsCode 

Dcouvrir les bugs sonar au plus tôt depuis l'IDE







## D/ Utilisation de sonar avec une bd externe 

- Avantages : scalabilité, persistence de données 


Prérequis pour le choix de la base

https://docs.sonarsource.com/sonarqube/latest/requirements/prerequisites-and-overview/



```

docker-compose.yml

version: '3'

services:
  sonarqube-postgresql:
    image: postgres:latest
    container_name: sonarqube-postgresql
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonarpassword
      POSTGRES_DB: sonarqubedb
      PGDATA: /var/lib/postgresql/data/pgdata

  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      SONARQUBE_JDBC_URL: jdbc:postgresql://sonarqube-postgresql:5432/sonarqubedb
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonarpassword
      SONAR_ES_BOOTSTRAP_CHECKS_DISABLE: true

```

`docker compose up`
