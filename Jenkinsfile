pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/tmp/flask-deployment'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Installing dependencies'
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests'
                sh '''
                    ./venv/bin/pytest -v test_app.py
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Flask application'
                sh '''
                    mkdir -p $DEPLOY_DIR
                    cp app.py requirements.txt $DEPLOY_DIR/
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application'
                sh '''
                    if [ -f $DEPLOY_DIR/app.pid ]; then
                        kill $(cat $DEPLOY_DIR/app.pid) 2>/dev/null || true
                    fi
                    cd $DEPLOY_DIR
                    JENKINS_NODE_COOKIE=dontKillMe \
                    nohup $WORKSPACE/venv/bin/python app.py \
                    > app.log 2>&1 &
                    echo $! > app.pid
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Verifying deployed application'
                sh '''
                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:5000/; then
                            exit 0
                        fi
                        sleep 2
                    done
                    echo "Application verification failed"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed and verified!'
        }
        failure {
            echo 'FAILURE: Pipeline execution failed!'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
