pipeline {

    agent none

    /*agent {
        docker {
            image 'maven:3-alpine'
            args '-v /home/blockchain/.m2:/root/.m2'
        }
    }*/

    environment {
        PROJECT_VERSION = '1.0' // readMavenPom().getVersion()
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /home/blockchain/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -B -DskipTests clean package'
                sh '>>>> Project version: ${env.PROJECT_VERSION}'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /home/blockchain/.m2:/root/.m2'
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
                    image 'maven:3-alpine'
                    args '-v /home/blockchain/.m2:/root/.m2'
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

