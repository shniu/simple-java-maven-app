pipeline {

    agent any

    /*agent {
        docker {
            image 'maven:3-alpine'
            args '-v /home/blockchain/.m2:/root/.m2'
        }
    }*/

    environment {
        MAVEN_JDK_IMAGE = 'maven:3.6-jdk-8'

        PROJECT_VERSION = readMavenPom(file: 'pom.xml').getVersion()
        PROJECT_ARITFACT_ID = readMavenPom(file: 'pom.xml').getArtifactId()
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    // image 'maven:3-alpine'
                    image "${env.MAVEN_JDK_IMAGE}"
                    reuseNode true
                    args '-v ${HOME}/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -B -DskipTests clean package'
                sh '>>>> java -version'
                echo ">>>> Project version: ${env.PROJECT_VERSION} and ${env.BRANCH_NAME} and ${PROJECT_ARITFACT_ID}"
            }
        }
        stage('Test') {
            agent {
                docker {
                    // image 'maven:3-alpine'
                    image "${env.MAVEN_JDK_IMAGE}"
                    reuseNode true
                    args '-v ${HOME}/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deliver') {
            agent {
                docker {
                    // image 'maven:3-alpine'
                    image "${env.MAVEN_JDK_IMAGE}"
                    reuseNode true
                    args '-v ${HOME}/.m2:/root/.m2'
                }
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }

        stage('Run docker') {
            agent {
                label 'ubuntu'
            }
            steps {
                sh 'docker run hello-world'
            }
        }
    }

    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}

