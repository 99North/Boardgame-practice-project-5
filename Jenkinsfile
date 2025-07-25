pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/99North/Boardgame-practice-project-5.git'
            }
        }
        
        stage('Code Compilation') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                // if you are already configure the sonarqube-server then only that server name is required
                withSonarQubeEnv('sonar') {
                    sh '''  $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=.  '''
                        
                    }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialID:  '----'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish To Nexus-Repo') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings-maven-pom_file', jdk: 'jdk11', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                        }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t systemail150/boardgame:latest .'
                        }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html  systemail150/boardgame:latest'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push systemail150/boardgame:latest'
                        }
                }
            }
        }
        
        stage('Deploy Into Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.36.192:6443') {
                    sh 'kubectl apply -f deployment-service.yaml'
                        }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.36.192:6443') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
        
        
        
        
    }
    
    
    post{
    always{
        script{
            def jobName=env.JOB_NAME
            def buildNumber=env.BUILD_NUMBER
            def pipelineStatus=currentBuild.result ?: "UNKNOWN"
            def bannerColor=pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
            
            def body = """
                <html>
                <body>
                <div style='border: 4px solid ${bannerColor}; padding: 10px;'>
                <h2 ${jobName} - Build ${buildNumber} </h2>
                <div style='background-color: ${bannerColor}; padding: 10px'>
                <h3 style="color:white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                </body>
                </html>
            """
            
            emailext(
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'systemmail150@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
                
                )
                
            
        
        }
    }
    }
    
    
    
    
}


