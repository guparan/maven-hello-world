pipeline {
    agent any	
	tools {
        maven "Maven 3.6.3"
    }	
	options {
        parallelsAlwaysFailFast()
    }    
    stages {
    
        stage('Parallel Stage') {
            parallel {                
                stage('Build the Maven project') {
                    steps {
                        sh "mvn clean install package"
                        archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                    }
                }
                stage('Trigger SonarQube') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh "mvn clean package sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL "
                        }
                    }
                }
            } // parallel
        } // stage('Parallel Stage')
        
        stage('Publish war file to Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'webapp/target/webapp.war']], mavenCoordinate: [artifactId: 'maven-project', groupId: 'com.example.maven-project', packaging: 'war', version: '1.1']]]
            }
        }
        
        stage('Build Docker image') {
            steps {
                sh '''
                    rm -Rf webapp.war
                    wget http://nexus:8081/repository/maven-releases/com/example/maven-project/maven-project/1.1/maven-project-1.1.war -O ${WORKSPACE}/webapp.war
                    docker build -t hello-world-afip:latest .
                '''
            }
        }
        
        stage('Run Docker container') {
            steps {
                sh '''
                    CONTAINER_NAME="hello-world-run"
                    OLD="$(docker ps --all --quiet --filter=name="$CONTAINER_NAME")"
                    if [ -n "$OLD" ]; then
                        docker rm -f $OLD
                    fi
                    docker run -d --name hello-world-run -p 18090:8080 hello-world-afip
                '''
            }
        }

    } // stages
} // pipeline
