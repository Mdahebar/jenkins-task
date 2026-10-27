pipeline {
    agent any

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        STAGING_DIR = '/var/www/staging'
        PROD_DIR    = '/var/www/prod'
        LIVE_DIR    = '/var/www/html'
        RELEASE_DIR = 'build_output'
    }

    stages {
        stage('Code Quality & Unit Test') {
            steps {
                echo "STAGE 1: Automated Validation..."
                sh '''
                    if [ ! -s index.html ]; then
                        echo "TEST FAILED: File is empty!"
                        exit 1
                    fi
                    if ! grep -q "<h1" index.html; then
                        echo "TEST FAILED: Missing <h1> tag!"
                        exit 1
                    fi
                    echo "TEST PASSED: Syntax verified."
                '''
            }
        }

        stage('Package Release Bundle') {
            steps {
                echo "STAGE 2: Archiving Artifact..."
                sh """
                    mkdir -p ${RELEASE_DIR}
                    tar -czf ${RELEASE_DIR}/release-v${BUILD_NUMBER}.tar.gz index.html
                """
                archiveArtifacts artifacts: "${RELEASE_DIR}/*.tar.gz", fingerprint: true
            }
        }

        stage('Deploy to Staging (QA)') {
            steps {
                echo "STAGE 3: Deploying automatically to Staging Environment..."
                sh """
                    cp -f index.html ${STAGING_DIR}/index.html
                    echo "Staging deployment complete."
                """
            }
        }

        stage('Production Safety Gate') {
            steps {
                // Pipeline execution yahan ruk jayegi aur human intervention ka wait karegi
                timeout(time: 5, unit: 'MINUTES') {
                    input message: "Staging verify ho gaya? Kya ise PRODUCTION par deploy karein?", 
                          ok: "Approve & Deploy to Prod",
                          submitter: "ahebar" // Sirf aapka account approve kar sakta hai
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "STAGE 4: Deploying to LIVE Production Server..."
                sh """
                    # Live Apache directory par deploy
                    cp -f index.html ${LIVE_DIR}/index.html
                    cp -f index.html ${PROD_DIR}/index.html
                    chmod 644 ${LIVE_DIR}/index.html
                    echo "PRODUCTION DEPLOYMENT SUCCESSFUL!"
                """
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Full deployment completed including production approval."
        }
        aborted {
            echo "SAFETY ABORT: Production deployment was rejected or timed out. Staging remains untouched."
        }
        failure {
            echo "ALERT: Pipeline failed before reaching production gate."
        }
    }
}
