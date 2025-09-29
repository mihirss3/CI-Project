pipeline {
  agent any
  options {
    skipDefaultCheckout()
    // add other options you want
  }
  environment {
    REPO = "mihirss3/CI-Project"   // update if repo path differs
  }
  stages {
    stage('Gate: require PR approval') {
      steps {
        script {
          // Must have CHANGE_ID for PR multibranch builds
          if (!env.CHANGE_ID) {
            echo "No CHANGE_ID (not a PR build). Skipping build."
            currentBuild.result = 'SUCCESS'
            return
          }

          // Use the username+password credential (username -> user, password -> PAT)
          withCredentials([usernamePassword(credentialsId: 'github-user-pat', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
            echo "Checking reviews for PR #${env.CHANGE_ID} in ${env.REPO}"
            def api = "https://api.github.com/repos/${env.REPO}/pulls/${env.CHANGE_ID}/reviews"
            def response = sh(script: "curl -s -H 'Authorization: token ${GITHUB_TOKEN}' '${api}'", returnStdout: true).trim()
            if (!response) {
              echo "Empty response from GitHub API; assuming no approval."
              currentBuild.result = 'SUCCESS'
              return
            }
            def reviews = readJSON text: response
            // consider any review with state APPROVED (case-insensitive)
            def approved = reviews.any { r -> (r.state ?: '').toString().toLowerCase() == 'approved' }
            if (!approved) {
              echo "No APPROVED review found for PR #${env.CHANGE_ID}. Skipping build."
              currentBuild.result = 'SUCCESS'
              return
            }
            echo "PR approved → proceeding with build."
          }
        }
      }
    }

    stage('Checkout') {
      steps {
        // checkout the PR source that Jenkins prepared for this multibranch job
        checkout scm
      }
    }

    stage('Setup Python') {
      steps {
        sh '''
          rm -rf venv
          python3 -m venv venv
          . venv/bin/activate
          python3 -m pip install --upgrade pip
          python3 -m pip install -r requirements.txt
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
      script {
        // only call junit if the results file exists
        def found = findFiles(glob: '**/test-results.xml')
        if (found.length > 0) {
          junit '**/test-results.xml'
        } else {
          echo "No test-results.xml found, skipping junit archive."
        }
      }
    }
  }
}
