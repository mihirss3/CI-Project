pipeline {
    agent any
    options {
        skipDefaultCheckout()  // we will checkout only if approved 
    }
    stages {
        stage('Gate on Review') {
            steps {
                script {
                    def action = env.GITHUB_EVENT_NAME
                    if (action != 'pull_request_review') {
                        error "Not a PR review event. Skipping build."
                    }
                    def payload = readJSON text: env.GITHUB_EVENT_PAYLOAD
                    if (payload.review.state != 'approved') {
                        error "PR review is not approved. Skipping build."
                    }
                    echo "PR approved – proceeding with build."
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
