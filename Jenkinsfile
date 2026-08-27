pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        APP_NAME = "curso-devops-lab3"
        SEMANTIC_VERSION = "1.0.0"
        
        // Reemplaza estos valores con tus datos reales si varían
        DOCKERHUB_USER = "tu-usuario-dockerhub" 
        GITHUB_USER = "nicomox777"       
        NAMESPACE = "cmarin"
    }

    stages {
        // 1.a Instalación de dependencias
        stage('a. Instalación de dependencias') {
            steps {
                sh 'npm ci'
            }
        }

        // 1.b Ejecución de pruebas y generación de cobertura
        stage('b. Ejecución de pruebas') {
            steps {
                sh 'npm run test'
                sh 'npm run test:cov'
            }
        }

        // 1.c Envío de cobertura a SonarQube y validación de puerta de calidad
    // 1.c Envío de cobertura a SonarQube y validación de puerta de calidad
     stage('c. SonarQube & Quality Gate') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh '''
                            npx sonar-scanner \
                              -Dsonar.projectKey=curso-devops-lab3 \
                              -Dsonar.projectName=curso-devops-lab3 \
                              -Dsonar.sources=src \
                              -Dsonar.tests=src \
                              -Dsonar.test.inclusions=**/*.spec.ts \
                              -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                              -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        // 1.d Build de la aplicación
        stage('d. Build de la aplicación') {
            steps {
                sh 'npm run build'
            }
        }

        // 1.e Construcción de imagen docker multistage
        stage('e. Construcción Docker Multistage') {
            steps {
                script {
                    dockerImage = docker.build("${APP_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        // 1.f Upload a Docker Hub
        stage('f. Upload Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        dockerImage.push("latest")
                        dockerImage.push("${SEMANTIC_VERSION}")
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }

        // 1.g Upload a GitHub Packages
        stage('g. Upload GitHub Packages') {
            steps {
                script {
                    docker.withRegistry('https://ghcr.io', 'github-credentials-id') {
                        dockerImage.push("latest")
                        dockerImage.push("${SEMANTIC_VERSION}")
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }

        // 1.h Actualización de imagen de Kubernetes local
        stage('h. Actualizar K8s Local') {
            steps {
                sh "kubectl set image deployment/nestjs-app nestjs-app=ghcr.io/${GITHUB_USER}/${APP_NAME}:${BUILD_NUMBER} -n ${NAMESPACE}"
            }
        }
    }
}