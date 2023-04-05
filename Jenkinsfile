pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('docker-credentials')
        }
        stages {
            stage('Build') {
                agent {
                    docker { image 'maven:3.6.3-openjdk-11-slim' }
                }
                    steps {
                      git credentialsId: 'github_token',
                      url: 'https://github.com/jimaspe/galaxy-examen-final-ascencio-jim.git',
                      branch: 'main'

                    sh 'mvn -B verify'
					sh 'ls -l target/'
                    }

                    post{
                        success {
                            archiveArtifacts artifacts: 'target/labmaven-*.jar', fingerprint: true, onlyIfSuccessful: true
                        }
                    }

            }
            stage('SonarQube') {
                agent {
                docker {
                    image 'maven:3.6.3-openjdk-11-slim'
                }
                }
                steps {
                script {
                    def scannerHome = tool 'scanner-default'
                    withSonarQubeEnv('sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=labmaven \
                                    -Dsonar.projectName=labmaven \
                                    -Dsonar.sources=src/main \
                                    -Dsonar.tests=src/test \
                                    -Dsonar.java.binaries=target/classes \
                    }
                }
                }
            }
            stage('Build Image') {
                steps {
                    copyArtifacts filter: 'target/*.jar',
                                    fingerprintArtifacts: true,
                                    projectName: '${JOB_NAME}',
                                    flatten: true,
                                    selector: specific('${BUILD_NUMBER}'),
                                    target: 'target'
                    sh 'docker --version'
                    sh 'docker-compose --version'
                    sh 'docker-compose build'
                }
            }
            stage('Publish Image') {
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker tag msmicroservice ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker push ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
            stage('Run Container') {
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker rm galaxyLab -f'
                        sh 'docker run -d -p 8080:8080 --name galaxyLab ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        //sh 'docker run -d -p 8080:8080 ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
        }
}