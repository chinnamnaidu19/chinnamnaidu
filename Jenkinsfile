pipeline {
    agent any
    tools {
        maven 'Maven 3'
    }
    stages {
        stage('Checkout') {
            steps {
                echo '1. Fetching source code from Git...'
                checkout scm
            }
        }
        stage('Fast Build') {
            steps {
                echo '2. Packaging MLRIT Dashboard WAR...'
                bat 'mvn clean package -DskipTests --batch-mode'
            }
        }
        stage('Deploy') {
            steps {
                echo '3. Copying WAR to Tomcat webapps...'
                bat 'copy /Y target\\mlritcollege.war "C:\\Program Files (x86)\\Apache Software Foundation\\Tomcat 9.0\\webapps\\"'
            }
        }
        stage('Verify Health') {
            steps {
                echo '4. Verifying deployment health...'
                powershell '''
                $url = "http://localhost:9090/mlritcollege/health.html"
                $retries = 10
                while ($retries -gt 0) {
                    try {
                        $resp = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 3
                        if ($resp.StatusCode -eq 200) {
                            Write-Host "Success: MLRIT Dashboard is Live!"
                            exit 0
                        }
                    } catch {
                        Start-Sleep -Seconds 2
                        $retries--
                    }
                }
                throw "Tomcat health check timed out."
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'src/main/webapp/index.html', allowEmptyArchive: false
        }
    }
}
