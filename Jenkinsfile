pipeline {

    agent any

    environment {
        APP_NAME        = 'pedes'
        CONTAINER_NAME  = 'pedes-hml'
        IMAGE_NAME      = 'localhost/pedes'
        HOST_PORT       = '8085'
        CONTAINER_PORT  = '80'
        DEPLOY_DIR      = '/u01/redelog_root/pedes_hml'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Codigo obtido pelo Jenkins atraves do SCM.'
            }
        }

        stage('Validar arquivos') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo " VALIDANDO PROJETO PEDES HML"
                    echo "=========================================="

                    test -f index.html
                    test -f style.css
                    test -f Dockerfile
                    test -d css
                    test -d js

                    echo "Arquivos principais encontrados."
                '''
            }
        }

        stage('Atualizar diretorio de deploy') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo " SINCRONIZANDO PEDES HML"
                    echo "=========================================="

                    test -d "${DEPLOY_DIR}"
                    test -w "${DEPLOY_DIR}"

                    rsync -av --delete \
                        --exclude='.git/' \
                        "$WORKSPACE/" \
                        "${DEPLOY_DIR}/"

                    echo ""
                    echo "Diretorio atualizado:"
                    echo "${DEPLOY_DIR}"
                '''
            }
        }

        stage('Build da imagem') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo " BUILD DA IMAGEM"
                    echo "=========================================="

                    cd "${DEPLOY_DIR}"

                    docker build \
                        -t "${IMAGE_NAME}:hml-${BUILD_NUMBER}" \
                        -t "${IMAGE_NAME}:hml" \
                        .

                    echo ""
                    echo "Imagem criada:"
                    docker images "${IMAGE_NAME}"
                '''
            }
        }

        stage('Parar container anterior') {
            steps {
                sh '''
                    echo "=========================================="
                    echo " REMOVENDO CONTAINER ANTERIOR"
                    echo "=========================================="

                    docker rm -f "${CONTAINER_NAME}" 2>/dev/null || true
                '''
            }
        }

        stage('Deploy HML') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo " DEPLOY PEDES HML"
                    echo "=========================================="

                    docker run -d \
                        --name "${CONTAINER_NAME}" \
                        -p "${HOST_PORT}:${CONTAINER_PORT}" \
                        "${IMAGE_NAME}:hml"

                    echo ""
                    echo "Container iniciado:"
                    docker ps --filter "name=${CONTAINER_NAME}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "=========================================="
                    echo " HEALTH CHECK"
                    echo "=========================================="

                    echo "Aguardando Nginx iniciar..."
                    sleep 3

                    echo "Testando HTTP..."

                    if curl -f --max-time 10 \
                        "http://localhost:${HOST_PORT}/"; then

                        echo ""
                        echo "=========================================="
                        echo " PEDES HML RESPONDEU HTTP COM SUCESSO"
                        echo "=========================================="

                    else

                        echo ""
                        echo "=========================================="
                        echo " FALHA NO HEALTH CHECK"
                        echo "=========================================="

                        echo ""
                        echo "===== STATUS ====="
                        docker ps -a \
                            --filter "name=${CONTAINER_NAME}" || true

                        echo ""
                        echo "===== LOGS ====="
                        docker logs "${CONTAINER_NAME}" 2>&1 || true

                        exit 1
                    fi
                '''
            }
        }
    }

    post {

        success {
            echo '''
==========================================
 PEDES HML DEPLOY REALIZADO COM SUCESSO
==========================================
'''
        }

        failure {
            echo '''
==========================================
 PEDES HML - FALHA NO DEPLOY
==========================================
'''
        }

        always {
            sh '''
                echo ""
                echo "===== STATUS FINAL DO CONTAINER ====="

                docker ps -a \
                    --filter "name=${CONTAINER_NAME}" || true

                echo ""
                echo "===== IMAGEM PEDES ====="

                docker images "${IMAGE_NAME}" || true
            '''
        }
    }
}
