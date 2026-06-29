pipeline {
    agent { label 'docker-maven-trivy'}

    tools {
        jdk 'jdk24'
        nodejs 'node26'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {

        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }

        stage ("Git Checkout") {
            steps {
                git 'https://github.com/u-niks/zomato-devsecops-deployment.git'
            }
        }

        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        stage("Code Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }

        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck(
                    odcInstallation: 'DP-Check',
                    additionalArguments: '''
                    --scan ./ 
                    --format XML 
                    --out .
                    --noupdate
                    '''
                )
                
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }

        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }

        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker tag zomato uniks/zomato:latest "
                        sh "docker push uniks/zomato:latest "
                    }
                }
            }
        }

        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'dockerhub'){
                       sh 'docker-scout quickview uniks/zomato:latest'
                       sh 'docker-scout cves uniks/zomato:latest'
                       sh 'docker-scout recommendations uniks/zomato:latest'
                   }
                }
            }
        }

        stage ("Deploy to Container") {
            steps {         
                sh '''
                docker rm -f zomato || true
                docker run -d --name zomato -p 3000:3000 uniks/zomato:latest
                '''
            }
        }
    }

    post {
    
        always {

            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                        <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                        </div>
                    </body>
                    </html>
                """,
                to: 'jadhavnikhil826@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy.txt'
        }
    }
}