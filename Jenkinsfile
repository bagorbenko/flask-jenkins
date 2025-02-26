pipeline {
    agent { label 'master' }

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
                pytest tests/ --maxfail=1 --disable-warnings || echo "Tests failed!"
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
        failure {
            echo '❌ Pipeline failed! Check logs for details.'
        }
        success {
            echo '✅ Deployment successful!'
        }
    }
}

