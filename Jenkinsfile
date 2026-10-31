
cat > Jenkinsfile <<'EOF'
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
                sh 'mvn -B -ntp clean verify'
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-local') {
                    sh '''
                        mvn -B -ntp \
                          org.sonarsource.scanner.maven:sonar-maven-plugin:5.3.0.6276:sonar \
                          -Dsonar.projectKey=java-devsecops-lab \
                          -Dsonar.projectName=java-devsecops-lab
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
            echo 'Java DevSecOps pipeline completed successfully!'
            echo 'Build, tests, SonarQube analysis and Quality Gate passed.'
        }

        failure {
            echo 'Pipeline failed. Please check Jenkins Console Output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
EOF

