pipeline{
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("Git Pull"){
            steps{
                git branch: 'main', url: 'https://github.com/Dinesh-Arivu/Uptime-Kuma-with-Real-Time-Downtime-Alerts-in-CI-CD-DevSecOps.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=uptimekuma \
                    -Dsonar.projectKey=uptimekuma '''
                }
            }
        }
        stage('Sonar-quality-gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t uptimekuma ."
                       sh "docker tag uptimekuma dinesh1097/uptimekuma:latest "
                       sh "docker push dinesh1097/uptimekuma:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image dinesh1097/uptimekuma:latest > trivy.json" 
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop uptimekuma | true"
                sh "docker rm uptimekuma | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name uptimekuma -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 dinesh1097/uptimekuma:latest'
            }
        }
    }
}
