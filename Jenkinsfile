pipeline {
    agent any

    options {
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        DEPLOY_TARGET = '/var/www/html'
        RELEASE_DIR   = 'build_output'
    }

    stages {
        stage('Code Verification & Inspection') {
            steps {
                echo "=========================================="
                echo "STAGE 1: Checking Workspace & Files"
                echo "=========================================="
                sh '''
                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html not found!"
                        exit 1
                    fi
                    echo "index.html exists. Previewing content:"
                    cat index.html
                '''
            }
        }

        stage('Automated Quality Tests') {
            steps {
                echo "=========================================="
                echo "STAGE 2: Running Automated Syntax Checks"
                echo "=========================================="
                sh '''
                    if [ ! -s index.html ]; then
                        echo "TEST FAILED: index.html is empty!"
                        exit 1
                    fi

                    if grep -q "<h1" index.html && grep -q "</h1>" index.html; then
                        echo "TEST PASSED: Valid HTML header tags detected."
                    else
                        echo "TEST FAILED: Required HTML <h1> tags missing!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Package Release Artifact') {
            steps {
                echo "=========================================="
                echo "STAGE 3: Packaging Release Bundle"
                echo "=========================================="
                sh """
                    mkdir -p ${RELEASE_DIR}
                    tar -czf ${RELEASE_DIR}/webapp-build-${BUILD_NUMBER}.tar.gz index.html
                    ls -lh ${RELEASE_DIR}/
                """
                archiveArtifacts artifacts: "${RELEASE_DIR}/*.tar.gz", fingerprint: true
            }
        }

        stage('Deploy to Web Server') {
            steps {
                echo "=========================================="
                echo "STAGE 4: Atomic Deployment to Apache"
                echo "=========================================="
                sh """
                    if [ -f ${DEPLOY_TARGET}/index.html ]; then
                        cp ${DEPLOY_TARGET}/index.html ${DEPLOY_TARGET}/index.html.bak
                    fi

                    cp -f index.html ${DEPLOY_TARGET}/index.html
                    chmod 644 ${DEPLOY_TARGET}/index.html
                """
            }
        }
    }

    post {
        always {
            echo "=========================================="
            echo "Pipeline finished execution for Build #${env.BUILD_NUMBER}"
            echo "Build URL: ${env.BUILD_URL}"
            echo "=========================================="
            cleanWs deleteDirs: true, notFailBuild: true
        }

        success {
            echo "STATUS: Build #${env.BUILD_NUMBER} successfully deploy ho chuki hai!"
            sh '''
                curl -X POST -H 'Content-type: application/json' \
                     --data "{\\"text\\":\\"SUCCESS: Build #${BUILD_NUMBER} deployed!\\"}" \
                     https://httpbin.org/post || true
            '''
        }

        failure {
            echo "ALERT: Pipeline execution fail hui Build #${env.BUILD_NUMBER} par!"
            sh '''
                curl -X POST -H 'Content-type: application/json' \
                     --data "{\\"text\\":\\"ALERT: Build #${BUILD_NUMBER} failed! Check console logs.\\"}" \
                     https://httpbin.org/post || true
            '''
        }
    }
}
