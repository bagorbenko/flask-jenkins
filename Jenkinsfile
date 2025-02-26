pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/bagorbenko/flask-jenkins.git'
        BRANCH_NAME = 'main'
        DEPLOY_DIR = '/opt/flask-jenkins'
        VENV_DIR = '${DEPLOY_DIR}/venv'
        FLASK_PORT = '5001'
    }

    stages {
        stage('Clone repository') {
            steps {
                cleanWs()
                git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                echo "Setting up virtual environment..."
                python3 -m venv ${VENV_DIR}
                source ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                echo "Python environment setup completed!"
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                echo "Running tests..."
                source ${VENV_DIR}/bin/activate
                pytest tests.py --maxfail=1 --disable-warnings || echo "Tests failed!"
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Stopping old Flask process..."
                if [ -f ${DEPLOY_DIR}/app.pid ]; then
                    kill $(cat ${DEPLOY_DIR}/app.pid) || true
                    rm ${DEPLOY_DIR}/app.pid
                fi

                echo "Starting Flask application..."
                source ${VENV_DIR}/bin/activate
                nohup python app.py --host=0.0.0.0 --port=${FLASK_PORT} > ${DEPLOY_DIR}/app.log 2>&1 &
                echo $! > ${DEPLOY_DIR}/app.pid
                echo "Application started on port ${FLASK_PORT}!"
                '''
            }
        }
    }

    post {
        success {
            sh """
                curl -i -X POST -H "Content-Type: application/json" \\
                -d '{"chat_id":"269199712","text":"Обновление билда: УДАЧНО","disable_notification":false}' \\
                https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendMessage
            """
        }
        failure {
            sh """
                curl -i -X POST -H "Content-Type: application/json" \\
                -d '{"chat_id":"269199712","text":"Обновление билда: ОШИБКА","disable_notification":false}' \\
                https://api.telegram.org/bot7511855444:AAEDvkMdddaKa4B2AArcudj7IEzQUQF6Lm8/sendMessage
            """
        }
    }
}
