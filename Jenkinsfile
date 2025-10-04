pipeline {
    agent any
      tools {
        sonarQubeScanner 'default-scanner'  
    }

    environment {
        DOCKER_HUB_REPO = 'bbabadara/exam-jenkins' 
        IMAGE_TAG = "latest" 
        RENDER_SERVICE_ID = 'srv-d378bs8gjchc73c28nb0' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm 
            }
        }

         // 🚨 Étape ajoutée pour  SonarQube
        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQubeLocal') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=test-jenkins \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=${sonar-token}
                    '''
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    def customImage = docker.build("${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub_credential') {
                        def image = docker.image("${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}")
                        image.push()
                        image.push('latest') 
                    }
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                script {
                    sh """
                        curl -X POST \
                            -H "Authorization: Bearer \${render_api}" \
                            -H "Content-Type: application/json" \
                            https://api.render.com/v1/services/\${RENDER_SERVICE_ID}/deploys
                    """
                }
            }
        }
    }

    post {
        always {
            sh script: 'docker rmi ${DOCKER_HUB_REPO}:${IMAGE_TAG} || true'
        }
        success {
            echo 'Pipeline réussi ! Image pushée et déployée sur Render.'
        }
        failure {
            echo 'Pipeline échoué. Vérifiez les logs.'
        }
    }
}