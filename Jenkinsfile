
 pipeline { 
    agent any 
     
    tools { 
        maven 'maven3' 
    } 
     
    environment { 
        SCANNER_HOME= tool 'sonar-scanner' 
    } 
 
    stages { 
        stage('Git Checkout') { 
            steps { 
                git branch: 'main', url: 'https://github.com/jaiswaladi246/Task-Master-Pro.git' 
            } 
        } 
         
        stage('Compile') { 
            steps { 
             sh  'mvn compile' 
            } 
        } 
         
        stage('Unit-Test') { 
            steps { 
             sh 'mvn test' 
            } 
        } 
         
        stage('Trivy FS Scan') { 
            steps { 
             sh 'trivy fs --format table -o fs.html .' 
            } 
        }
 
        stage('Sonar Analysis') { 
            steps { 
                withSonarQubeEnv('sonar') { 
                    sh ''' $SCANNER_HOME/bin/sonar-scanner 
Dsonar.projectName=taskmaster -Dsonar.projectKey=taskmaster \ 
                    -Dsonar.java.binaries=target ''' 
                } 
            } 
        } 
         
        stage('Build Application') { 
            steps { 
             sh 'mvn package' 
            } 
        } 
         
        stage('Publish Artifact') { 
            steps { 
             withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', 
maven: 'maven3', mavenSettingsConfig: '', traceability: true) { 
                    sh 'mvn deploy' 
                } 
            } 
        } 
         
        stage('Docker Build & Tag') { 
            steps { 
                script { 
             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { 
                    sh 'docker build -t bittush8789/taskmaster:latest .' 
                } 
            } 
        }} 
         
        stage('Trivy Image Scan') { 
            steps { 
             sh 'trivy image --format table -o image.html bittush8789/taskmaster:latest' 
            } 
        } 
         
        stage('Docker Push') { 
            steps { 
                script { 
             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { 
                    sh 'docker push bittush8789/taskmaster:latest' 
                } 
            } 
        }} 
         
        stage('K8 Deploy') { 
            steps { 
             withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', 
contextName: '', credentialsId: 'k8-token', namespace: 'webapps', 
restrictKubeConfigAccess: false, serverUrl: 
'https://1DC375532F6FB38A39069BFC0460C894.gr7.ap-south
1.eks.amazonaws.com') { 
                    sh 'kubectl apply -f deployment-service.yml' 
                    sleep 30 
                } 
            } 
        } 
         
        stage('Verify K8 Deployment') { 
            steps { 
             withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', 
contextName: '', credentialsId: 'k8-token', namespace: 'webapps', 
restrictKubeConfigAccess: false, serverUrl: 
'https://1DC375532F6FB38A39069BFC0460C894.gr7.ap-south
1.eks.amazonaws.com') { 
                    sh 'kubectl get pods -n webapps' 
                    sh 'kubectl get svc -n webapps' 
                    
                } 
            } 
        } 
    } 
 post { 
    always { 
        script { 
 
            def jobName = env.JOB_NAME 
            def buildNumber = env.BUILD_NUMBER 
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN' 
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red' 
 
            def body = """ 
                <html> 
                <body> 
                <div style="border: 4px solid ${bannerColor}; padding: 10px;"> 
                <h2>${jobName} - Build ${buildNumber}</h2> 
                <div style="background-color: ${bannerColor}; padding: 10px;"> 
                <h3 style="color: white;">Pipeline Status: 
${pipelineStatus.toUpperCase()}</h3> 
                </div> 
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p> 
                </div> 
                </body> 
                </html> 
            """ 
 
            emailext ( 
                subject: "${jobName} - Build ${buildNumber} - 
${pipelineStatus.toUpperCase()}", 
                body: body, 
                to: 'bittush9534@gmail.com', 
                from: 'jenkins@example.com', 
                replyTo: 'jenkins@example.com', 
                mimeType: 'text/html', 
                attachmentsPattern: 'trivy-image-report.html' 
            ) 
        } 
    } 
} 
 
}