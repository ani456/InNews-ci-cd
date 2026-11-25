pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        githubPush()
    }

    environment {
        DEPLOY_HOST     = '18.216.31.98'
        DEPLOY_USER     = 'deploy'
        APP_DIR         = '/opt/project2/innews'
        APP_VENV        = '/opt/project2/innews/venv'
        SSH_CREDENTIALS = 'innews-app-ssh'   // Jenkins credential ID
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Downloading source code from GitHub...'
                checkout scm
            }
        }

        stage('Setup Python') {
            steps {
                sh '''
                    python3 --version
                    python3 -m venv venv
                '''
            }
        }

        stage('Setup Python') {
    steps {
        echo 'Setting up Python virtual environment...'

        sh '''
            set -e

            echo "Checking Python..."
            whoami
            python3 --version

            echo "Removing previous virtual environment..."
            rm -rf venv

            echo "Creating virtual environment..."
            python3 -m venv venv

            echo "Verifying pip..."
            venv/bin/python -m pip --version

            echo "Python setup completed successfully."
        '''
    }
}

        stage('Deploy to EC2') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' || env.BRANCH_NAME == 'main' }
            }
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes $DEPLOY_USER@$DEPLOY_HOST \
                            "mkdir -p $APP_DIR"

                        rsync -az \
                            --exclude='.git/' \
                            --exclude='venv/' \
                            --exclude='.env' \
                            --exclude='__pycache__/' \
                            -e "ssh -o StrictHostKeyChecking=yes" \
                            ./ $DEPLOY_USER@$DEPLOY_HOST:$APP_DIR/

                        ssh -o StrictHostKeyChecking=yes $DEPLOY_USER@$DEPLOY_HOST \
                            "test -d $APP_VENV || python3 -m venv $APP_VENV
                             $APP_VENV/bin/pip install -r $APP_DIR/requirements.txt &&
                             sudo -n systemctl restart innews"
                    '''
                }
            }
        }

        stage('Health Check') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' || env.BRANCH_NAME == 'main' }
            }
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes $DEPLOY_USER@$DEPLOY_HOST \
                            "sudo -n systemctl is-active --quiet innews &&
                             curl --fail --retry 10 --retry-delay 3 --retry-connrefused \
                             http://127.0.0.1:8501/_stcore/health"
                    '''
                }
            }
        }
    }

    post {
        success { echo 'InNews deployed successfully!' }
        failure { echo 'Deployment failed. Check console output.' }
        always  { echo 'Pipeline execution completed.' }
    }
}