pipeline {
    agent any
    stages {
        stage ('Build BackEnd') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=a4f89da6d7abf99c3e276697bbe2f11a38d8e56b -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                // sleep(10)
                // timeout(time: 1, unit: 'MINUTES') {
                //     waitForQualityGate abortPipeline: true
                // }

                sh 'echo Quality Gate'
            }
        }
        stage ('Deploy BackEnd') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'login_tomcat', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Tests') {
            steps {
                dir('api-test') {
                    git credentialsId: 'gitHub_login', url: 'https://github.com/carlos-george/tasks-api-tests'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('tasks-frontend') {

                    git credentialsId: 'gitHub_login', url: 'https://github.com/carlos-george/tasks-frontend'
                    sh 'mvn clean package -DskipTests=true'
                    deploy adapters: [tomcat8(credentialsId: 'login_tomcat', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Tests') {
            steps {
                dir('func-test') {
                    git credentialsId: 'gitHub_login', url: 'https://github.com/carlos-george/tasks-functional-tests'
                    sh 'mvn test'
                }
                // sh 'echo Functional tests'
            }
        }
        stage ('Deploy Prod') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(10) 
                dir('func-test') {
                    sh 'mvn verify -Dskip.surefire.tests'
                }
                // sh 'echo Health Check'
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, func-test/target/surefire-reports/*.xml, func-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, tasks-frontend/target/tasks.war', followSymlinks: false, onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below.', subject: 'Build $BUILD_NUMBER has failed', to: 'bolinha248+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below.', subject: 'Build is fine', to: 'bolinha248+jenkins@gmail.com'
        }
    }
}


