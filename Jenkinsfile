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
                  # start fresh so ownership is correct
                  rm -rf venv
                  python3 -m venv venv
                  . venv/bin/activate

                  # force install into the venv, never into ~/.local
                  pip install --upgrade pip
                  pip install --no-user -r requirements.txt
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

    }

    post {
        always {
            junit '**/test-results.xml'
        }
    }
}
