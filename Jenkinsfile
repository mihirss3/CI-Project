pipeline {
    agent any
    options {
        skipDefaultCheckout()  // we will checkout only if approved 
    }
    stages {
        stage('Check Approval') {
            steps {
                script {
                    // Always print env once to verify variables
                    sh 'env | sort'

                    // Use GitHub API to check approval
                    def token = credentials('github-token-id')   // store a PAT in Jenkins creds
                    def prNumber = env.CHANGE_ID
                    def repo   = "mihirss3/CI-Project"
                    def response = sh(
                        script: "curl -s -H 'Authorization: token ${token}' https://api.github.com/repos/${repo}/pulls/${prNumber}/reviews",
                        returnStdout: true
                    )
                    def reviews = readJSON text: response
                    def approved = reviews.any { it.state == 'APPROVED' }

                    if (!approved) {
                        echo "No approval yet. Stopping pipeline."
                        currentBuild.result = 'SUCCESS'
                        return
                    }
                }
            }
        }
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
