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

                rsync -rlt \
                    --no-owner \
                    --no-group \
                    --no-perms \
                    --delete \
                    --exclude='.git/' \
                    "$WORKSPACE/" \
                    ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/

                echo ""
                echo "=========================================="
                echo " ARQUIVOS SINCRONIZADOS COM SUCESSO"
                echo "=========================================="

                echo ""
                echo "Destino:"
                echo "${DEPLOY_HOST}:${DEPLOY_DIR}"
            '''
        }
    }
}
