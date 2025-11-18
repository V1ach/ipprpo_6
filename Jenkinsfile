pipeline {
    agent any

    tools {
        maven 'Maven 3.9.11'
        jdk 'JDK 21'
    }

    stages {
        stage('Diagnose Project Structure') {
            steps {
                echo "=== CHECKING PROJECT STRUCTURE ==="
                bat 'dir /S /B'
                bat 'if exist pom.xml (echo "✅ pom.xml exists" && type pom.xml) else (echo "❌ pom.xml NOT FOUND")'
                bat 'if exist src (echo "✅ src exists" && dir src /S /B) else (echo "❌ src NOT FOUND")'
                bat 'if exist src\\main (echo "✅ src/main exists" && dir src\\main /S /B) else (echo "❌ src/main NOT FOUND")'
                bat 'if exist src\\test (echo "✅ src/test exists" && dir src\\test /S /B) else (echo "❌ src/test NOT FOUND")'
            }
        }

        stage('Check Java Version') {
            steps {
                echo "=== CHECKING JAVA VERSION ==="
                bat 'java -version'
                bat 'javac -version'
            }
        }

        stage('Build') {
            steps {
                echo "=== BUILDING PROJECT ==="
                bat 'mvn -v'
                bat 'mvn -B clean compile -X'  // -X для детального лога
                bat 'if exist target (echo "✅ target created" && dir target /B) else (echo "❌ target NOT created")'
            }
        }

        stage('Test') {
            steps {
                echo "=== TESTING PROJECT ==="
                bat 'mvn -B test -X'  // -X для детального лога
                bat 'if exist target (echo "✅ target exists" && dir target /B) else (echo "❌ target STILL not created")'
                bat 'if exist target\\surefire-reports (echo "✅ surefire-reports exists" && dir target\\surefire-reports /B) else (echo "❌ surefire-reports NOT exists")'
                bat 'if exist target\\site (echo "✅ site exists" && dir target\\site /B) else (echo "❌ site NOT exists")'
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
    }

    post {
        always {
            echo "=== BUILD FINISHED ==="
        }
    }
}