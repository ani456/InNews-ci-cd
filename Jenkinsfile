
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
        DEPLOY_HOST     = '3.12.83.169'
        DEPLOY_USER     = 'deploy'
        APP_DIR         = '/opt/project2/innews'
        APP_VENV        = '/opt/project2/innews/venv'
        SSH_CREDENTIALS = 'innews-app-ssh'
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
                echo 'Setting up Python virtual environment...'

                sh '''
                    set -e

                    whoami
                    python3 --version

                    rm -rf venv

                    python3 -m venv venv

                    venv/bin/python -m pip --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'

                sh '''
                    set -e

                    test -f requirements.txt
                    test -f App.py

                    venv/bin/python -m pip install --upgrade pip

                    venv/bin/python -m pip install -r requirements.txt

                    venv/bin/python -m compileall -q App.py

                    echo "Dependencies installed successfully."
                '''
            }
        }

        stage('Deploy to EC2') {
            when {
                expression {
                    env.GIT_BRANCH == 'origin/main' ||
                    env.BRANCH_NAME == 'main'
                }
            }

            steps {
                echo 'Deploying application to EC2...'

                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh '''
                        set -e

                        ssh -o StrictHostKeyChecking=yes \
                            $DEPLOY_USER@$DEPLOY_HOST \
                            "mkdir -p $APP_DIR"

                        rsync -az \
                            --exclude='.git/' \
                            --exclude='venv/' \
                            --exclude='.venv/' \
                            --exclude='.env' \
                            --exclude='__pycache__/' \
                            -e "ssh -o StrictHostKeyChecking=yes" \
                            ./ $DEPLOY_USER@$DEPLOY_HOST:$APP_DIR/

                        ssh -o StrictHostKeyChecking=yes \
                            $DEPLOY_USER@$DEPLOY_HOST \
                            "set -e
                             test -d $APP_VENV || python3 -m venv $APP_VENV
                             $APP_VENV/bin/python -m pip install --upgrade pip
                             $APP_VENV/bin/python -m pip install -r $APP_DIR/requirements.txt
                             sudo -n systemctl restart innews"
                    '''
                }
            }
        }

        stage('Health Check') {
            when {
                expression {
                    env.GIT_BRANCH == 'origin/main' ||
                    env.BRANCH_NAME == 'main'
                }
            }

            steps {
                echo 'Checking application health...'

                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh '''
                        set -e

                        ssh -o StrictHostKeyChecking=yes \
                            $DEPLOY_USER@$DEPLOY_HOST \
                            "sudo -n systemctl is-active --quiet innews &&
                             curl --fail --retry 10 --retry-delay 3 \
                             --retry-connrefused \
                             http://127.0.0.1:8501/_stcore/health"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'InNews deployed successfully!'
        }

        failure {
            echo 'Deployment failed. Check console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
