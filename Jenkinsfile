pipeline {
    agent any

    options {
        // Build 10 minute se zyada chale toh kill kar do
        timeout(time: 10, unit: 'MINUTES')
        // Hard drive full hone se bachane ke liye sirf last 10 builds preserve karo
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Do builds ek sath overlap na hon
        disableConcurrentBuilds()
        // Har console line ke sath exact execution timestamp print karo
        timestamps()
    }

    triggers {
        // Har 1 minute me GitHub repository poll karega
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
                        echo "ERROR: index.html not found in repository!"
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
                    # Test 1: File size check (empty file detection)
                    if [ ! -s index.html ]; then
                        echo "TEST FAILED: index.html is completely empty!"
                        exit 1
                    fi

                    # Test 2: Structure verification
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
                echo "STAGE 3: Packaging Compressed Release Bundle"
                echo "=========================================="
                sh """
                    mkdir -p ${RELEASE_DIR}
                    tar -czf ${RELEASE_DIR}/webapp-build-${BUILD_NUMBER}.tar.gz index.html
                    ls -lh ${RELEASE_DIR}/
                """
                // Jenkins build history me archive save karna
                archiveArtifacts artifacts: "${RELEASE_DIR}/*.tar.gz", fingerprint: true
            }
        }

        stage('Deploy to Web Server') {
            steps {
                echo "=========================================="
                echo "STAGE 4: Atomic Deployment to Apache"
                echo "=========================================="
                sh """
                    # Backup existing deployment
                    if [ -f ${DEPLOY_TARGET}/index.html ]; then
                        cp ${DEPLOY_TARGET}/index.html ${DEPLOY_TARGET}/index.html.bak
                    fi

                    # Deploy latest artifact
                    cp -f index.html ${DEPLOY_TARGET}/index.html
                    chmod 644 ${DEPLOY_TARGET}/index.html
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline run completed for Build #${BUILD_NUMBER}."
        }
        success {
            echo "STATUS: Build #${BUILD_NUMBER} DEPLOYED SUCCESSFULLY TO PRODUCTION!"
        }
        failure {
            echo "ALERT: Build #${BUILD_NUMBER} FAILED! Review test logs immediately."
        }
    }
}
