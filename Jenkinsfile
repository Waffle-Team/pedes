pipeline {

    agent any

    environment {
        APP_NAME = 'pedes'
        CONTAINER_NAME = 'pedes-hml'
        IMAGE_NAME = 'localhost/pedes'
        HOST_PORT = '8085'
        CONTAINER_PORT = '80'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Código já foi obtido pelo Jenkins.'
            }
        }

        stage('Validar arquivos') {
            steps {
                sh '''
                    set -e

                    echo "===== VALIDANDO PROJETO PEDES ====="

                    test -f index.html
                    test -f style.css
                    test -d css
                    test -d js

                    echo "Arquivos principais encontrados."
                '''
            }
        }

        stage('Build da imagem') {
            steps {
                sh '''
                    set -e

                    echo "===== BUILD DA IMAGEM ====="

                    docker build \
                        -t ${IMAGE_NAME}:hml-${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:hml \
                        .
                '''
            }
        }

        stage('Parar container anterior') {
            steps {
                sh '''
                    echo "===== PARANDO CONTAINER ANTERIOR ====="

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true
                '''
            }
        }

        stage('Deploy HML') {
            steps {
                sh '''
                    set -e

                    echo "===== SUBINDO PEDES HML ====="

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        ${IMAGE_NAME}:hml

                    echo "Container iniciado."
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "===== AGUARDANDO NGINX ====="

                    sleep 3

                    echo "===== TESTANDO HTTP ====="

                    curl -f http://localhost:${HOST_PORT}/

                    echo ""
                    echo "PEDES HML respondeu HTTP com sucesso."
                '''
            }
        }
    }

    post {

        success {
            echo '=========================================='
            echo ' PEDES HML DEPLOY REALIZADO COM SUCESSO'
            echo '=========================================='
        }

        failure {
            echo '=========================================='
            echo ' PEDES HML - FALHA NO DEPLOY'
            echo '=========================================='
        }

        always {
            sh '''
                echo "===== STATUS DO CONTAINER ====="
                docker ps -a --filter name=${CONTAINER_NAME} || true
            '''
        }
    }
}
