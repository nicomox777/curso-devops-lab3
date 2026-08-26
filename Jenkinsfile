pipeline {
    agent any

    environment {
        APP_NAME = "curso-devops-lab3"
        DOCKERHUB_USER = "tu-usuario-dockerhub"
        GITHUB_USER = "tu-usuario-github"
        SEMANTIC_VERSION = "1.0.0"
        NAMESPACE = "tuinicialyapellido" // Ej: cmarin
    }

    stages {
        stage('Instalar Dependencias') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Ejecutar Pruebas') {
            steps {
                sh 'npm run test'
                sh 'npm run test:cov'
            }
        }

        stage('SonarQube & Quality Gate') {
            steps {
                // Requiere Plugin de SonarQube en Jenkins y servidor activo
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'npm run sonar' // Asegúrate de tener script sonar o sonar-scanner
                }
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build de Aplicación') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Imagen Docker') {
            steps {
                script {
                    dockerImage = docker.build("${APP_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push a Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push("latest")
                        dockerImage.push("${SEMANTIC_VERSION}")
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Push a GitHub Packages') {
            steps {
                script {
                    docker.withRegistry('https://ghcr.io', 'github-credentials') {
                        dockerImage.push("latest")
                        dockerImage.push("${SEMANTIC_VERSION}")
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Despliegue a Kubernetes Local') {
            steps {
                // 2.e Aplicar manifiesto
                sh 'kubectl apply -f kubernetes.yaml'
                
                // 1.h Actualización dinámica con build number
                sh "kubectl set image deployment/nestjs-app nestjs-app=ghcr.io/${GITHUB_USER}/${APP_NAME}:${BUILD_NUMBER} -n ${NAMESPACE}"
                
                // 2.e Validar ejecucion de pods
                sh "kubectl rollout status deployment/nestjs-app -n ${NAMESPACE}"
            }
        }
    }
}