# De Jenkins Maven/Freestyle à Pipeline

Adrien Lecharpentier

> Depuis Novembre 2014, Jenkins connait un nouveau type de job: Pipeline. Encore peu utilisé, j'essayerai de vous montrer à quoi il correspond et quelles problématiques il résout. Pour cela, je vous montrerai la migration d'un job type Maven classique vers un Pipeline et comment aller encore plus loin avec avec Pipeline-as-Code.

[Vidéo](https://www.youtube.com/watch?v=lLc1MY9Bpuk)

`In Progress`

Avec l'arrivée de git, les branches se sont démultipliées. Définir un ou plusieurs jobs Jenkins par branche est fastidieux et contre productif. Les pipeline permettent de régler cette problématique (et d'autres mais c'est essentiellement celle-ci qui m'intéresse pour le moment).

Les pipeline permettent de définir un build jenkins sous forme de code groovy. Ce code peut-être saisi directement dans l'ihm jenkins ou tout simplement commité avec le code source dans un fichier `Jenkinsfile`.

Sous forme de fichier, il est alors possible de créer un job de type `Multibranch Pipeline` qui va créer automatiquement un job par branche contenant ce fichier.

Il devient alors possible de valider des branches de fonctionnalités sans modifier la configuration Jenkins.


L'exemple suivant effectue les étapes suivantes :
- checkout de la branche
- build avec gradle en utilisant la jdk-8u65 définie dans la configuration Jenkins
- fournit le jar résultant en téléchargement via l'ihm jenkins
- construit une image docker
- la pousse dans un registry

```groovy
node {
    stage 'Checkout'
    checkout scm

    stage 'Build'
    withJava {
      sh 'bash gradlew build'
    }

    dir('build/libs') {
        archive "trombi-back.jar"
    }

    stash name: 'binary', includes: "build/libs/trombi-back.jar"
    dir('src/main/docker') {
        stash name: 'dockerfile', includes: 'Dockerfile'
    }
}

node('docker') {
    unstash 'dockerfile'
    unstash 'binary'

    stage 'Building Docker Image'
    image = docker.build("trombi:${env.BRANCH_NAME}")
}

node('docker') {
    stage 'Publishing Docker Image'
    // requirement: local docker registry available on port 5000
    docker.withRegistry('http://monregistry:5000', '') {
        image.push("${env.BRANCH_NAME}")
    }
}




// Custom step
def withJava(def body) {
    def javaHome = tool name: 'jdk-8u65', type: 'hudson.model.JDK'

    withEnv(["JAVA_HOME=${javaHome}"]) {
        body.call()
    }
}
```

Autres avantages :
- on peut modifier la version de java sans impacter les autres build ni modifier la configuration des jobs dans jenkins
- on peut spécialiser les noeuds jenkins et ne les utiliser que pour une portion spécifique du build (docker par exemple)
