pipeline {
    agent any   
    tools {
        maven "Maven 3.6.3"
    }   
    options {
        parallelsAlwaysFailFast()
    }    
    stages {
        
        stage('Clean the Maven project') {
            steps {
                sh "mvn clean"
            }
        }
    
        stage('Parallel stage 1') {
            parallel {
                stage('Build the Maven project') {
                    steps {
                        sh "mvn package"
                        archiveArtifacts(
                            artifacts: '**/*.war', 
                            followSymlinks: false
                        )
                    }
                }
                stage('Trigger SonarQube') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh "mvn sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL"
                        }
                    }
                }
            } // parallel
        } // stage('Parallel stage 1')
        
        stage('Parallel stage 2') {
            parallel {
                stage('Push webapp to Nexus') {
                    steps {
                        nexusPublisher(
                            nexusInstanceId: 'Nexus', 
                            nexusRepositoryId: 'maven-releases', 
                            packages: [[
                                $class: 'MavenPackage', 
                                mavenAssetList: [[
                                    classifier: '', 
                                    extension: '', 
                                    filePath: 'webapp/target/webapp.war'
                                ]], 
                                mavenCoordinate: [
                                    artifactId: 'maven-project', 
                                    groupId: 'com.example.maven-project', 
                                    packaging: 'war', 
                                    version: "${env.GIT_COMMIT}"
                                ]
                            ]]
                        )
                    }
                }
        
                stage('Create new Docker image for webapp') {
                    steps {
                        sh '''
                            rm -Rf webapp.war
                            cp webapp/target/webapp.war .
                            docker build -t hello-world-afip:latest .
                        '''
                    }
                }
            } // parallel
        } // stage('Parallel stage 2')
        
        stage('Run Docker container based on new image') {
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
        
        stage('Load testing with JMeter') {
            steps {
                sh '''
                    jmeter \
                        -Jjmeter.save.saveservice.output_format=xml \
                        -Jjmeter.save.saveservice.response_data.on_error=true \
                        -n -t JMeterTestPlan.jmx  \
                        -l testresult.jlt
                '''
                perfReport(
                    filterRegex: '', 
                    showTrendGraphs: true, 
                    sourceDataFiles: 'testresult.jlt'
                )
            }
        }

    } // stages
} // pipeline
