pipeline {

    agent {
        label 'ubuntu'
    }

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

        // Just demo
        PWD_DIR = sh(returnStdout: true, script: 'pwd').trim()
        // 可以当做 CI 环境下构建镜像时的版本号
        LATEST_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()

        HOSTS = [
                "192.168.1.42",
                "192.168.1.38",
                "192.168.1.131"
        ]
        HELLO_WORLD = 'hello-world'
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
                sh 'java -version'
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
            steps {
                script {
                    for (host in HOSTS) {
                        sshRemoteDockerPull(host, env.HELLO_WORLD)
                    }
                }
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

def sshRemoteDockerPull(host, imageTag) {
    def remote = [:]
    remote.name = host
    remote.host = host
    remote.allowAnyHosts = true

    withCredentials([usernamePassword(credentialsId: 'remoteNodeAccount', usernameVariable: 'username', passwordVariable: 'password')]) {
        remote.user = username
        remote.password = password

        // sshCommand(remote: remote, command: 'for i in {1..5}; do echo -n \"Loop \$i \"; date ; sleep 1; done')

        writeFile file: 'abc.sh', text: """
            docker pull ${imageTag}
            docker images
        """
        sshScript remote: remote, script: 'abc.sh'
    }
}

