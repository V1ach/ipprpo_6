pipeline {
    agent any

    tools {
        maven 'Maven-3.9.11'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }

    environment {
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=false'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Fix Git Permissions') {
            steps {
                bat 'git config --global --add safe.directory C:/Users/Vlach/Documents/GitHub/ipprpo_6'
                bat 'git config --global --add safe.directory C:/Users/Vlach/Documents/GitHub/ipprpo_6/.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn -v'
                bat 'mvn -B -U clean compile'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn -B test'
                // Диагностические команды
                bat 'dir target /B'
                bat 'if exist target\\surefire-reports (dir target\\surefire-reports /B) else (echo "surefire-reports not found")'
                bat 'if exist target\\site (dir target\\site /B) else (echo "site not found")'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    publishHTML([reportDir: 'target/site/jacoco',
                                reportFiles: 'index.html',
                                reportName: 'JaCoCo Coverage',
                                keepAll: true,
                                alwaysLinkToLastBuild: true,
                                allowMissing: false])
                }
            }
        }

        stage('Package') {
            steps {
                bat 'mvn -B package'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Quality gates') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                }
            }
            steps {
                echo 'Quality checks placeholder'
            }
        }

        stage('Deploy (local)') {
            when {
                branch 'main'
            }
            steps {
                bat 'java -jar target/java-maven-ci-demo-1.0.0.jar'
                echo 'Application deployed locally'
            }
        }
    }

    post {
        success {
            echo "Build successful for ${env.BRANCH_NAME}"
        }
        failure {
            echo "Build failed for ${env.BRANCH_NAME}"
        }
    }
}