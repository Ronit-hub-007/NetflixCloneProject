pipeline {
    agent any
    tools{
        jdk 'JDK17'
        nodejs 'NODEJS16'
    }
    environment {
        SCANNER_HOME=tool 'Sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps{
                cleanWs()
            }
        }
        stage("Sonarqube Analysis ") {
            steps{
                withSonarQubeEnv('Sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate") {
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
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
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps{
                script{
                        withDockerRegistry(credentialsId: 'DockerHubCreds', toolName: 'docker') {
                            sh "docker build --build-arg TMDB_V3_API_KEY=554141f0d7debfb2c976e06fcf8b58bf -t netflixclone ."
                            sh "docker tag netflixclone rohtmore007/netflixclone:latest "
                            sh "docker push rohtmore007/netflixclone:latest "
                            sh "docker push rohtmore007/netflixclone:V${BUILD_NUMBER} "
                        }
                }
            }
        }
        stage("TRIVY") {
            steps{
                sh "trivy image rohtmore007/netflixclone:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container') {
            steps{
                sh 'docker run -d --name netflix -p 80:80 sevenajay/netflix:latest'
            }
        }
    }
    post {
        always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'rmore4889@gmail.com,thesagarshah33@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}