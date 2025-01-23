
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    enviornment {
        SCANNER_HOME= tool 'sonar'
        MINIKUBE_CLUSTER_NAME = 'minikube'
        DEPLOYMENT_YML = '/home/zignuts/boardgame/deployment-service.yml'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-credentials', url: 'https://github.com/Mangesh4150/Boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
        }
               
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {

                script { 
                    withDockerRegistry(credentialsId: 'jenkins-docker-cred', toolName: 'docker') {
                        sh "docker build -t mangesh4150/boardshack:latest ."
}
                }
                   
               }
            }
        
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html mangesh4150/boardshack:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'jenkins-docker-cred', toolName: 'docker') {
                        sh "docker push mangesh4150/boardshack:latest"
                    }
               }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                script {
                    // Apply the updated deployment and service YAML files to Minikube
                    sh 'kubectl apply -f ${DEPLOYMENT_YML} --validate=false'
                    sh 'kubectl apply -f service.yml --validate=false'
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
                script {
                    // Apply the updated deployment and service YAML files to Minikube
                    sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
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
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'v.mangesh1718@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

    }
}
}

