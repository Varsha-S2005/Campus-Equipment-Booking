
pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:/usr/bin:/bin"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Java and Maven') {
            steps {
                sh '''
                    echo "Checking Java version..."
                    java -version

                    echo "Checking Java compiler..."
                    javac -version

                    echo "Checking Maven version..."
                    mvn -version
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean package'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'target/surefire-reports/*.xml'
        }

        success {
            archiveArtifacts artifacts: 'target/*.jar',
                             fingerprint: true
        }
    }
}
