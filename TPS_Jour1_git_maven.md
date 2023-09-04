# Guide pour les TPs git et maven du jour 1

  

## A/ Prérequis :

  

### i/ Connexion à la VM

  

Dans le fichier `~/.ssh/config` ajouter cette section :

  

```plaintext

Host <username>

HostName <ip>

User <username>

LocalForward 8080 localhost:8080

LocalForward 9000 localhost:9000

```

  

Remplacez `<username>` par votre nom d'utilisateur de la VM.

  

Puis depuis le terminal :

  

```shell

ssh <username>

```

  

Si OK, connectez l'IDE VScode à votre machine en ssh.

  

Sous VScode : Remote explorer (menu de gauche) + sélectionner la machine `<username>`

  

### ii/ Forker et cloner le projet du TP

  

[Projet à forker et cloner](https://github.com/Zerofiltre-Courses/zerofiltre-blog-backend-inow)

  

En Forkant, décochez "cloner la branche main uniquement" car on va travailler sur une branche différente.

  

Ajoutez votre clé publique à github.

  

Dans la VM :

  

```shell

cat  ~/.ssh/<username>_inow_ed25519

```

  

Copiez le contenu

  

Ajoutez à github: github profile ==> settings, ssh+gpg keys

  

Clonez

  

```shell

cd  java

```

  

Clonez le projet :

  

```shell

git  clone  git@github.com:YOUR_GITHUB_USERNAME/zerofiltre-blog-backend-inow.git

```

  

## D/ Mise en place du scénario + explications

  

Mettez-vous sur la branche `email-on_article_in_review_inow_jour1` :

  

```shell

git  checkout  email-on_article_in_review_inow_jour1

```

  

Créez une nouvelle branche et déplacez-vous dessus :

  

```shell

git  checkout  -b  email-on_article_in_review_inow-conflict-a

```

  

Exécutez `mvn test` => échec

  

Corrigez en ajoutant ces deux lignes :

  

```java

UserActionEvent  userActionEvent = new  UserActionEvent(appUrl, Locale.forLanguageTag(author.getLanguage()), author, "", existingArticle, Action.ARTICLE_SUBMITTED);

notificationProvider.notify(userActionEvent);

```

  

Exécutez à nouveau `mvn test` => OK

  

Poussez sur le repo distant et faites la pull request vers `email-on_article_in_review_inow_jour1` qui est acceptée

  

Revenez à la branche `email-on_article_in_review_inow_jour1`

  

Créez une nouvelle branche et déplacez-vous dessus :

  

```shell

git  checkout  -b  email-on_article_in_review_inow-conflict-b

```

  

Corrigez en ajoutant ces deux lignes :

  

```java

UserActionEvent  userActionEvent = new  UserActionEvent(appUrl, Locale.forLanguageTag("FR-fr"), author, "any", existingArticle, Action.ARTICLE_SUBMITTED);

notificationProvider.notify(userActionEvent);

```

  

Puisqu'il y a des modifications sur la branche principale :

  

```shell

git  checkout  email-on_article_in_review_inow

git  pull

git  checkout  -

```

  

Sur `email-on_article_in_review_inow-conflict-b`, faites un rebase de `email-on_article_in_review_inow_jour1`

  

Résolvez le conflit en fusionnant les modifications à la main ou sur Theia IDE

  

```shell

git  add  .

git  rebase  --continue

git  status

git  push  -f

```

  

Car la branche locale `email-on_article_in_review_inow-conflict-b` a divergé de la branche distante `origin/email-on_article_in_review_inow-conflict-b`

  

La branche locale est la plus à jour, donc on force l'écrasement de la branche distante.

  

### La release :

  

```shell

git  checkout  -b  release-1.0.0

git  push  -u  origin  release-1.0.0

```

  

Mise en prod manuelle

  

Création de tag :

  

```shell

git  tag  -a  v1.7.0  -m  "Fix Send email on article review"

git  push  -u  origin  --tags

```

  

Supprimez la branche de release

  

## E/ Revenir à l’état de la branche distante

  

```shell

git  reset  --hard  origin/branch_name

```

  

## F/ Corriger une catastrophe

  

```shell

git  reflog

```

  

Copiez le hash

  

```shell

git  reset  --hard  hash

```

  

# ======= Maven =======

  

## A/ TP : Observer les cycles de vie à l’oeuvre

  

Rappel : théorie

  

Clonez le projet : [lien vers le projet](https://github.com/Zerofiltre-Courses/maven-quickstart)

  

```shell

git  clone  git@github.com:Zerofiltre-Courses/maven-quickstart.git

```

  

```shell

mvn  clean  install

```

  

## B/ Génération et exploration d'un Projet multi module

  

```shell

mvn  archetype:generate  \

-DgroupId=tech.zerofiltre \

-DartifactId=sample-multimodule \

-Dversion=1.0-SNAPSHOT \

-DarchetypeGroupId=org.apache.maven.archetypes \

-DarchetypeArtifactId=maven-archetype-quickstart \

-DarchetypeVersion=1.4 \

-Dpackaging=pom

```

  

```shell

cd  sample-multimodule

```

  

Éditez le fichier `pom.xml` et ajoutez `<packaging>pom</packaging>` en dessous de `<version>`

  

```shell

mvn  archetype:generate  \

-DgroupId=tech.zerofiltre.common \

-DartifactId=common \

-Dversion=1.0-SNAPSHOT \

-DarchetypeGroupId=org.apache.maven.archetypes \

-DarchetypeArtifactId=maven-archetype-quickstart \

-DarchetypeVersion=1.4

```

  

```shell

mvn  archetype:generate  \

-DgroupId=tech.zerofiltre.services \

-DartifactId=services \

-Dversion=1.0-SNAPSHOT \

-DarchetypeGroupId=org.apache.maven.archetypes \

-DarchetypeArtifactId=maven-archetype-quickstart \

-DarchetypeVersion=1.4

```

  

```shell

mvn  archetype:generate  \

-DgroupId=tech.zerofiltre.web \

-DartifactId=web \

-Dversion=1.0-SNAPSHOT \

-DarchetypeGroupId=org.apache.maven.archetypes \

-DarchetypeArtifactId=maven-archetype-quickstart \

-DarchetypeVersion=1.4

```

  

Analysons le contenu du fichier `pom.xml` via VS code

  

## C/ DependencyManagement et PluginManagement dans un projet multimodule

  

### Gestion centralisée versions JUNIT via DependencyManagement

  

Ouvrir la classe `AppTest.java` de `common`

  

1er constat, les annotations junit 4 sont encore utilisées, nous voulons la 5.

  

Ajoutez dans la section `dependencies` du pom parent par celle-ci :

  

```xml

<dependencyManagement>

<dependencies>

<!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->

<dependency>

<groupId>org.junit.jupiter</groupId>

<artifactId>junit-jupiter-api</artifactId>

<version>5.10.0</version>

<scope>test</scope>

</dependency>

</dependencies>

</dependencyManagement>

```

  

En supprimant les dépendances junit 4 dans `common`, nous avons un souci.

  

Il faut mentionner explicitement la dépendance dans `common`, SANS la version comme suit :

  

```xml

<dependency>

<groupId>org.junit.jupiter</groupId>

<artifactId>junit-jupiter-api</artifactId>

</dependency>

```

  

## D/ PluginManagement et configuration de quelques plugin

  

### i/ Exploration

  

Dans le pom parent et fils, les plugins se répètent

  

- Supprimez les sections `<pluginManagement>` dans les fils

  

- Supprimez les sections `properties

  

` des fils qui seront héritées du parent

  

- Remplacez la dépendance junit par celle-ci, tel que l'on a fait dans `common`

  

```xml

<!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine -->

<dependency>

<groupId>org.junit.jupiter</groupId>

<artifactId>junit-jupiter-engine</artifactId>

<version>5.10.0</version>

<scope>test</scope>

</dependency>

```

  

### ii/ Configuration du plugin surefire pour l'exécution des tests

  

Ouvrir la classe `App.java` de `common`, `service` et `web` et créer la méthode `testMe()`

  

```java

boolean  testMe(String value){

return  "theGoodValue".equals(value);

}

```

  

Ouvrir la classe `AppTest.java` de `common`, `service` et `web`, corriger les imports et ajouter la méthode :

  

```java

@Test

void  testMe_shouldBe_True() {

App  app = new  App();

assertTrue(app.testMe("theGoodValue"));

}

```

  

### iii/ Configuration du plugin JACOCO (Java Code coverage)

  

Il est utilisé pour mesurer à quel point les lignes de code du projet sont testées.

  

Il va calculer les pourcentages de couverture, les afficher dans un rapport détaillé sous forme de pages web.

  

Les pourcentages calculés peuvent être utilisés pour imposer un seuil minimum à l’application

  

Ajouter le plugin au parent

  

```xml

<plugin>

<groupId>org.jacoco</groupId>

<artifactId>jacoco-maven-plugin</artifactId>

<version>0.8.10</version>

</plugin>

```

  

Ceci n'est pas suffisant, il faut ajouter :

  

```xml

<executions>

<execution>

<goals>

<!-- exécute le goal prepare-agent du plugin lors de la phase par défaut définie par le dev du plugin -->

<goal>prepare-agent</goal>

</goals>

</execution>

<execution>

<id>report</id>

<phase>test</phase>

<goals>

<!-- exécute le goal report (génération de rapports) du plugin lors de la phase test du cycle de vie maven -->

<goal>report</goal>

</goals>

</execution>

</executions>

```

  

Ce n'est que lorsqu'on fera `mvn test` que les rapports seront générés

  

Comme nous l'avons fait avec les dépendances, chaque module fils se doit d'indiquer au parent qu'il souhaite utiliser le dit plugin en le déclarant

  

```xml

<project>

...

<build>

<plugins>

<plugin>

<groupId>org.jacoco</groupId>

<artifactId>jacoco-maven-plugin</artifactId>

</plugin>

</plugins>

</build>

...

</project>

```

  

`mvn clean test` à la racine du projet

  

Les dossiers `target/site` seront générés dans chaque projet.

  

Clic droit télécharger afin de l'ouvrir dans la machine locale (eh oui nous sommes en ssh)

  

Une fois téléchargé ouvrir `index.html`

  

Imposer un seuil :

  

Ajouter dans la config du plugin dans le pom parent

  

```xml

<plugin>

<executions>

...

<execution>

<id>jacoco-check</id>

<!-- Pas besoin de préciser une phase, c'est rattaché à la phase verify par défaut -->

<goals>

<goal>check</goal>

</goals>

<configuration>

<rules>

<rule>

<element>PACKAGE</element>

<limits>

<limit>

<counter>LINE</counter>

<value>COVEREDRATIO</value>

<minimum>0.70</minimum>

</limit>

</limits>

</rule>

</rules>

</configuration>

</execution>

...

</executions>

</plugin>

```

  

À la racine du projet :

  

```shell

mvn  clean  verify  ==> qui  va  exécuter  test  dans  la  foulée (cycle de  vie  default,  vous  vous  rappelez  ?  https://zerofiltre.tech/articles/183)

```

  

Échec car l'on a 50% chez `common`

  

Passer à 50% (0.5) et `mvn clean site` de nouveau

  

### iv/ Configuration du plugin javadoc

  

Si vous codez une application à destination d’un large public, il se peut que vous ayez besoin d’exposer la javadoc.

  

pom parent :

  

```xml

<plugin>

<groupId>org.apache.maven.plugins</groupId>

<artifactId>maven-javadoc-plugin</artifactId>

<version>3.5.0</version>

<executions>

<execution>

<id>report</id>

<!-- exécute le goal javadoc (génération de la javadoc) pendant la phase package -->

<phase>package</phase>

<goals>

<goal>javadoc</goal>

</goals>

</execution>

</executions>

</plugin>

```

  

pom fils :

  

```xml

<plugin>

<groupId>org.apache.maven.plugins</groupId>

<artifactId>maven-javadoc-plugin</artifactId>

</plugin>

```

  

Puis `mvn clean package`, s'il ne trouve pas JAVA_HOME

  

```shell

echo  'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc

source  ~/.bashrc

```

  

Téléchargez le dossier site, puis ouvrez `site/aipdocs/index.html`

  

### v/ Configuration du plugin site

  

Site représente le site de la documentation technique autogénérée du projet JAVA

  

Faites `mvn clean site` à la racine du projet.

  

3 dossiers de site sont générés.

  

Vous pouvez les télécharger et les ouvrir via `index.html`.

  

À ce stade, nous avons les rapports jacoco, javadoc et site qui sont produits mais sont séparés

  

Nous allons les agréger grâce à la config reporting qui est une exécution rattachée à la phase site du cycle de vie.

  

Comment fonctionne le reporting ?

  

```xml

<build>

<reporting>

<plugins>

<plugin>

<groupId><!-- Plugin Group ID --></groupId>

<artifactId><!-- Plugin Artifact ID --></artifactId>

<version><!-- Plugin Version --></version>

<reportSets>

<reportSet>

<id><!-- Report Set ID --></id>

<reports>

<report><!-- GOAL de REPORTING DU PLUGIN --></report>

</reports>

</reportSet>

<!-- Additional <reportSet> elements -->

</reportSets>

</plugin>

</plugins>

</reporting>

</build>

```

  

Nous voulons :

  

- Agréger les javadoc du projet en un seul dossier

  

site

  

```xml

<reporting>

<plugins>

<plugin>

<groupId>org.apache.maven.plugins</groupId>

<artifactId>maven-javadoc-plugin</artifactId>

<version>3.5.0</version>

</plugin>

</plugins>

</reporting>

```

  

Attention : les rapports individuels seront encore générés lors de la phase `package`.

  

Faites `mvn clean site`

  

Téléchargez `site` de la racine et ouvrez `index.html`

  

Cliquez sur projet Report => JavaDoc : ===> les 3 packages

  

- Agréger les rapports jacoco du projet en un seul dossier site

  

```xml
<plugin>

<groupId>org.jacoco</groupId>

<artifactId>jacoco-maven-plugin</artifactId>

<version>0.8.10</version>

<reportSets>

<reportSet>

<reports>

<report>report-aggregate</report>

</reports>

</reportSet>

</reportSets>

</plugin>
```
