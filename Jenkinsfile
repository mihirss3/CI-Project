pipeline {
    agent any

    options {
        skipDefaultCheckout()
        timestamps()
    }

    environment {
        REPO = "mihirss3/CI-Project"
        GITHUB_CREDENTIALS = "github-user-pat"
    }

    stages {
        stage('Gate: require PR approval') {
            steps {
                script {
                    // Skip if not a PR build
                    if (!env.CHANGE_ID) {
                        echo "Not a PR build. Skipping pipeline."
                        currentBuild.result = 'SUCCESS' // mark as success
                        return
                    }

                    // Check GitHub reviews for approval
                    withCredentials([usernamePassword(
                        credentialsId: env.GITHUB_CREDENTIALS, 
                        usernameVariable: 'GITHUB_USER', 
                        passwordVariable: 'GITHUB_TOKEN')]) {

                        def api = "https://api.github.com/repos/${env.REPO}/pulls/${env.CHANGE_ID}/reviews"
                        def response = sh(
                            script: "curl -s -H 'Authorization: token ${GITHUB_TOKEN}' '${api}'",
                            returnStdout: true
                        ).trim()

                        def reviews = readJSON text: response
                        def approved = reviews.any { r -> (r.state ?: '').toString().toLowerCase() == 'approved' }

                        if (!approved) {
                            echo "PR not approved yet. Skipping tests."
                            currentBuild.result = 'SUCCESS' // mark as success
                            return
                        } else {
                            echo "PR approved. Continuing build..."
                        }
                    }
                }
            }
        }

        stage('Checkout') {
            when {
                expression { env.CHANGE_ID && currentBuild.result == null }
            }
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: env.CHANGE_BRANCH ?: 'main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: "https://github.com/${env.REPO}.git",
                        credentialsId: env.GITHUB_CREDENTIALS
                    ]]
                ])
            }
        }

        stage('Setup Python') {
            when {
                expression { env.CHANGE_ID && currentBuild.result == null }
            }
            steps {
                sh '''
                  rm -rf venv
                  python3 -m venv venv
                  . venv/bin/activate
                  python3 -m pip install --upgrade pip
                  pip install --no-user -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            when {
                expression { env.CHANGE_ID && currentBuild.result == null }
            }
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
            script {
                def testFiles = findFiles(glob: 'test-results.xml')
                if (testFiles.length > 0) {
                    junit 'test-results.xml'
                }
            }
        }
    }
}
