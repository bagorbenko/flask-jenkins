pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/bagorbenko/flask-jenkins.git'
        BRANCH_NAME = 'main'
        DEPLOY_DIR = '/opt/flask-jenkins'
        VENV_DIR = "${DEPLOY_DIR}/venv"
    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Target environment (dev/prod)')
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout and Update Code') {
            steps {
                script {
                    sh """
                    if [ ! -d ${DEPLOY_DIR} ]; then
                        sudo mkdir -p ${DEPLOY_DIR}
                        sudo chown -R jenkins:jenkins ${DEPLOY_DIR}
                        git clone -b ${BRANCH_NAME} ${REPO_URL} ${DEPLOY_DIR}
                    else
                        cd ${DEPLOY_DIR}
                        git reset --hard
                        git pull origin ${BRANCH_NAME}
                    fi
                    """
                }
            }
        }

        stage('Setup Python Environment') {
            steps {
                script {
                    sh """
                    if [ ! -d ${VENV_DIR} ]; then
                        echo "Creating virtual environment..."
                        python3 -m venv ${VENV_DIR}
                    fi

                    echo "Activating virtual environment and installing requirements..."
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r ${DEPLOY_DIR}/requirements.txt
                    """
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh """
                    echo "Running tests..."
                    . ${VENV_DIR}/bin/activate
                    pytest ${DEPLOY_DIR}/tests.py --maxfail=1 --disable-warnings
                    """
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    expression { return params.ENV == 'dev' }
                    allOf {
                        expression { return params.ENV == 'prod' }
                        expression { return env.BRANCH_NAME == 'main' }
                    }
                }
            }
            steps {
                script {
                    sh """
                    echo "Restarting Flask service via systemd..."
                    sudo systemctl restart flask-jenkins
                    echo "Service restarted successfully!"
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                def lastCommit = sh(script: "cd ${DEPLOY_DIR} && git log -1 --pretty=format:'%h'", returnStdout: true).trim()
                def author = sh(script: "cd ${DEPLOY_DIR} && git log -1 --pretty=format:'%an'", returnStdout: true).trim()
                def branch = sh(script: "cd ${DEPLOY_DIR} && git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                def statusEmoji = buildStatus == 'SUCCESS' ? "✅" : "❌"

                def message = """$statusEmoji Обновление билда: $buildStatus
Last commit: $lastCommit
Author: $author
Branch: $branch
Environment: ${params.ENV}
"""

                sh """
                    curl -i -X POST -H "Content-Type: application/json" \\
                    -d '{
                        "chat_id": "269199712",
                        "text": "$message",
                        "disable_notification": false
                    }' \\
                    https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendMessage
                """
            }
        }
    }
}
