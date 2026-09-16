pipeline {

    agent any

    environment {
        APP_NAME          = 'pedes'
        CONTAINER_NAME    = 'pedes-hml'
        IMAGE_NAME        = 'localhost/pedes'
        HOST_PORT         = '8085'
        CONTAINER_PORT    = '80'
        DEPLOY_DIR        = '/u01/redelog_root/pedes_hml'

        DEPLOY_HOST       = '10.11.80.199'
        DEPLOY_USER       = 'jenkins'
        SSH_CREDENTIAL    = 'ssh-199'
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

        stage('Sincronizar arquivos') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        set -e

                        echo "=========================================="
                        echo " SINCRONIZANDO PEDES HML"
                        echo "=========================================="

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "test -d ${DEPLOY_DIR}"

                        rsync -av --delete \
                            --exclude='.git/' \
                            "$WORKSPACE/" \
                            ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/

                        echo ""
                        echo "Diretorio sincronizado:"
                        echo "${DEPLOY_DIR}"
                    '''
                }
            }
        }

        stage('Build da imagem') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        set -e

                        echo "=========================================="
                        echo " BUILD DA IMAGEM"
                        echo "=========================================="

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "
                            cd ${DEPLOY_DIR} &&
                            podman build \
                                -t ${IMAGE_NAME}:hml-${BUILD_NUMBER} \
                                -t ${IMAGE_NAME}:hml \
                                .
                            "

                        echo ""
                        echo "Imagem criada com sucesso."
                    '''
                }
            }
        }

        stage('Parar container anterior') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        echo "=========================================="
                        echo " REMOVENDO CONTAINER ANTERIOR"
                        echo "=========================================="

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "podman rm -f ${CONTAINER_NAME} 2>/dev/null || true"
                    '''
                }
            }
        }

        stage('Deploy HML') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        set -e

                        echo "=========================================="
                        echo " DEPLOY PEDES HML"
                        echo "=========================================="

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "
                            podman run -d \
                                --name ${CONTAINER_NAME} \
                                -p ${HOST_PORT}:${CONTAINER_PORT} \
                                ${IMAGE_NAME}:hml
                            "

                        echo ""
                        echo "Container iniciado com sucesso."
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        set -e

                        echo "=========================================="
                        echo " HEALTH CHECK"
                        echo "=========================================="

                        sleep 3

                        ssh -o StrictHostKeyChecking=no \
                            ${DEPLOY_USER}@${DEPLOY_HOST} \
                            "
                            curl -f --max-time 10 \
                                http://localhost:${HOST_PORT}/
                            "

                        echo ""
                        echo "=========================================="
                        echo " PEDES HML RESPONDEU HTTP COM SUCESSO"
                        echo "=========================================="
                    '''
                }
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
    }
}
