pipeline {
    agent any

    // We skip the default checkout until we know the PR is approved
    options {
        skipDefaultCheckout()
    }

    stages {

        stage('Check Approval') {
            steps {
                // Expose the GitHub token as an env var for curl
                withCredentials([string(credentialsId: 'github-token-id', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Sanity check that we’re running inside a PR context
                        if (!env.CHANGE_ID) {
                            error "CHANGE_ID is not set – this must be a pull-request build."
                        }

                        echo "Checking approval for PR #${env.CHANGE_ID}"

                        // Query GitHub reviews for this PR
                        def apiUrl = "https://api.github.com/repos/mihirss3/CI-Project/pulls/${env.CHANGE_ID}/reviews"
                        def json = sh(
                            script: "curl -s -H 'Authorization: token ${GITHUB_TOKEN}' ${apiUrl}",
                            returnStdout: true
                        ).trim()

                        def reviews = readJSON text: json
                        def approved = reviews.any { it.state == 'APPROVED' }

                        if (!approved) {
                            echo "No GitHub approval yet. Stopping pipeline."
                            currentBuild.result = 'SUCCESS'
                            // End the build early without marking it as failure
                            error("PR not approved")
                        }
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                // In a multibranch job, this checks out the correct
                // merge commit or PR head automatically.
                checkout scm
            }
        }

        stage('Setup Python') {
            steps {
                sh '''
                    rm -rf venv
                    python3 -m venv venv
                    . venv/bin/activate
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
                    echo "Running tests..."
                    pytest --maxfail=1 --disable-warnings -v --junitxml=test-results.xml
                '''
            }
        }
    }
}
