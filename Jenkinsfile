pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/mihirss3/CI-Project.git', branch: 'main'
            }
        }

        stage('Setup Python') {
            steps {
                sh '''
                  python3 -m venv venv
                  . venv/bin/activate
                  pip install --upgrade pip
                  pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    export PYTHONPATH=$PYTHONPATH:$(pwd)
                    echo "Running tests from project root..."
                    pytest --maxfail=1 --disable-warnings -v --junitxml=test-results.xml
                '''
            }
        }

        stage('Build') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    sh './build.sh'
                }
            }
        }
    }

    post {
        always {
            junit '**/test-results.xml' // optional, for test reporting
        }
    }
}
