pipeline {
    agent {
        label 'java21'
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {

        stage('Prepare Build') {
            steps {
                echo 'Preparing Java 21 Maven build environment'

                sh '''
                    java -version
                    mvn -version
                    git --version
                '''
            }
        }

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Fail Fast Validation') {
            steps {
                sh '''
                    test -f pom.xml
                    test -d src/main/java
                    test -d src/test/java

                    echo "Source validation successful"
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    mvn -B -ntp clean verify
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Java DevSecOps build completed successfully!'
        }

        failure {
            echo 'Java DevSecOps build failed. Check console output.'
        }
    }
}
