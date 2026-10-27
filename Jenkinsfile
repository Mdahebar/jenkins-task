pipeline {
    agent any

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '15'))
        disableConcurrentBuilds()
        timestamps()
    }

    parameters {
        choice(
            name: 'DEPLOY_ACTION', 
            choices: ['DEPLOY', 'ROLLBACK'], 
            description: 'Naya code deploy karna hai ya purana version rollback karna hai?'
        )
        string(
            name: 'ROLLBACK_VERSION', 
            defaultValue: '', 
            description: 'Agar ROLLBACK chuna hai, toh purana Build Number likhein (e.g., 1 ya 2)'
        )
    }

    environment {
        LIVE_DIR    = '/var/www/html'
        RELEASE_DIR = 'build_output'
    }

    stages {
        stage('Code Quality Check') {
            when {
                expression { params.DEPLOY_ACTION == 'DEPLOY' }
            }
            steps {
                echo "STAGE 1: Code Verification & Automated Tests..."
                sh '''
                    if [ ! -s index.html ]; then
                        echo "TEST FAILED: File empty hai!"
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

        stage('Package Release Artifact') {
            when {
                expression { params.DEPLOY_ACTION == 'DEPLOY' }
            }
            steps {
                echo "STAGE 2: Archiving New Release Package..."
                sh """
                    mkdir -p ${RELEASE_DIR}
                    tar -czf ${RELEASE_DIR}/webapp-build-${BUILD_NUMBER}.tar.gz index.html
                """
                archiveArtifacts artifacts: "${RELEASE_DIR}/*.tar.gz", fingerprint: true
            }
        }

        stage('Deploy New Version') {
            when {
                expression { params.DEPLOY_ACTION == 'DEPLOY' }
            }
            steps {
                echo "STAGE 3: Deploying New Build #${BUILD_NUMBER} to Live Apache..."
                sh """
                    cp -f index.html ${LIVE_DIR}/index.html
                    chmod 644 ${LIVE_DIR}/index.html
                    echo "DEPLOYMENT COMPLETE: Build #${BUILD_NUMBER} is now live."
                """
            }
        }

        stage('Execute Instant Rollback') {
            when {
                expression { params.DEPLOY_ACTION == 'ROLLBACK' }
            }
            steps {
                echo "STAGE: Rolling back to Build #${params.ROLLBACK_VERSION}..."
                sh """
                    # Input validation
                    if [ -z "${params.ROLLBACK_VERSION}" ]; then
                        echo "ERROR: Rollback ke liye Build Number specify karna mandatory hai!"
                        exit 1
                    fi

                    TARGET_ARTIFACT="/var/lib/jenkins/jobs/${JOB_NAME}/builds/${params.ROLLBACK_VERSION}/archive/${RELEASE_DIR}/webapp-build-${params.ROLLBACK_VERSION}.tar.gz"

                    if [ ! -f "\$TARGET_ARTIFACT" ]; then
                        echo "ERROR: Artifact nahi mila at \$TARGET_ARTIFACT"
                        exit 1
                    fi

                    # Instant restoration
                    tar -xzf "\$TARGET_ARTIFACT" -C ${LIVE_DIR}/
                    chmod 644 ${LIVE_DIR}/index.html
                    echo "SUCCESS: Rollback to Build #${params.ROLLBACK_VERSION} completed instantly!"
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished execution. Action taken: ${params.DEPLOY_ACTION}"
        }
        success {
            echo "STATUS: Pipeline action executed successfully!"
        }
        failure {
            echo "ALERT: Pipeline execution failed. Verify logs."
        }
    }
}
