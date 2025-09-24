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
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    echo "Running tests from project root..."
                    pytest --maxfail=1 --disable-warnings -v
                '''
            }
        }
    }

    post {
        always {
            junit '**/test-results.xml' // optional, for test reporting
        }
    }
}
