// Hello-world build pipeline
pipeline {
    agent any

    tools {
        maven "Maven 3.6.3"
    }
    stages {
        stage('Run the maven build') {
           steps {
              sh "mvn clean install package"
           }
        }
    }
}
