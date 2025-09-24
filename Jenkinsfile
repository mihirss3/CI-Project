pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/<your-username>/<your-repo>.git', branch: 'main'
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
                sh '. venv/bin/activate && pytest tests/'
            }
        }
    }

    post {
        always {
            junit '**/test-results.xml' // optional, for test reporting
        }
    }
}
