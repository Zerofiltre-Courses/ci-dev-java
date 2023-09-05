# Guide pour les TPs du jour 2

## Test driven development

Le but sera de mettre l'accent sur des baby steps pour découvrir l'implémentation de l'algorithme de la feature


### 1/ Implémenter une feature en TDD avec Junit

#### Gestion d'un panier de courses : ShippingCart
Le panier de course devrait permettre aux clients:
- d'ajouter et de retirer des éléments : CartItem
- de calculer le prix total du panier
- d'appliquer une ristourne de 10 % si le panier a plus de 10 éléments et la somme totale > 150 €

	
	

### 2/ Implémenter une feature en TDD avec Junit et Mockito

Reprenons le projet Zerofiltre, déplacez-vous sur la branche: email-on_article_in_review_inow_jour2

[Feature à implémenter](https://trello.com/c/lyFfoaJy/407-restreindre-lacc%C3%A8s-aux-articles-par-id)  



## Releases et snapshots maven

### 1/ Exploration du fichier settings.xml 

`nano /etc/maven/settings.xml`



### 2/ Déployer le jar sur un repo maven distant

Créer un compte sur [artifactory cloud](https://jfrog.com/fr/start-free/) => choisir la version cloud => utiliser aws => suivre la configuration accompagnée



#### i. Déploiement de la snapshot

La configuration générée par artifactory devrait ressembler à ceci: 

````xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd" xmlns="http://maven.apache.org/SETTINGS/1.2.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <servers>
    <server>
      <username>${env.ARTIFACTORY_SERVER_USERNAME}</username>
      <password>${env.ARTIFACTORY_SERVER_PASSWORD}</password>
      <id>central</id>
    </server>
    <server>
      <username>${env.ARTIFACTORY_SERVER_USERNAME}</username>
      <password>${env.ARTIFACTORY_SERVER_PASSWORD}</password>
      <id>snapshots</id>
    </server>
  </servers>

  <profiles>
    <profile>
	
      <repositories>
	  
        <repository>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <id>central</id>
          <name>libs-release</name>
          <url>https://zerofiltre.jfrog.io/artifactory/libs-release</url>
        </repository>
		
        <repository>
          <snapshots />
          <id>snapshots</id>
          <name>libs-snapshot</name>
          <url>https://zerofiltre.jfrog.io/artifactory/libs-snapshot</url>
        </repository>
		
      </repositories>
	  
      <pluginRepositories>
	  
        <pluginRepository>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <id>central</id>
          <name>libs-release</name>
          <url>https://zerofiltre.jfrog.io/artifactory/libs-release</url>
        </pluginRepository>
		
        <pluginRepository>
          <snapshots />
          <id>snapshots</id>
          <name>libs-snapshot</name>
          <url>https://zerofiltre.jfrog.io/artifactory/libs-snapshot</url>
        </pluginRepository>
		
      </pluginRepositories>
	  
      <id>artifactory</id>
	  
    </profile>

  </profiles>

  <activeProfiles>
    <activeProfile>artifactory</activeProfile>
  </activeProfiles>

</settings>


<distributionManagement>
    <snapshotRepository>
        <id>snapshots</id>
        <name>a0ufk52h2bit9-artifactory-primary-0-snapshots</name>
        <url>https://zerofiltre.jfrog.io/artifactory/libs-snapshot</url>
    </snapshotRepository>
</distributionManagement>

````


Définissez les variables d'environnement nécessaires: 

```
echo "export ARTIFACTORY_SERVER_USERNAME=<nom_utilisateur_genere>" >> ~/.bashrc
echo "export ARTIFACTORY_SERVER_PASSWORD=<mot_de_passe_genere>" >> ~/.bashrc

source ~/.bashrc
```



Puis :

 `mvn clean deploy -DskipIntegrationTests -s .m2/settings.xml`
 
Le  deploy sera utilisé.


#### ii. Déploiement de la release

Nous allons déployer une release de notre produit

Définir la version de l'artefact à configurer : 

`mvn versions:set -DnewVersion=0.0.1`


Puis créer un repository de releases sous jfrog : artifacts ==> set me up ==> libs-release => Deploy : copier le repository et l'ajouter dans la section `distributionManagement`


````xml
<repository>
        <id>central</id>
        <name>a0ufk52h2bit9-artifactory-primary-0-releases</name>
        <url>https://zerofiltre.jfrog.io/artifactory/libs-release</url>
</repository>
````

Puis: `mvn clean deploy -DskipTests -s .m2/settings.xml`




#### iii. Télécharger les deps depuis maven central si le repo entreprise pas dispo


Il faut définir le plugin repository et repository maven central à la main dans la partie `profile`

```xml

       <repository>
          <id>central</id>
          <url>https://repo.maven.apache.org/maven2</url>
        </repository>

		   <pluginRepository>
          <id>central</id>
          <url>https://repo.maven.apache.org/maven2</url>
        </pluginRepository> 

```
        















































































