pipeline {
    agent any 
    
    tools {
        dockerTool 'docker'
    }
    
    triggers { githubPush() } 

    environment {
        GITHUB_REPO = 'git@github.com:valentin-provost/pipeline-as-code.git'
        APP_NAME = 'pipeline-as-code'
        GITHUB_USER = 'Jenkins CI'
        GITHUB_EMAIL = 'jenkins@ci.local'
        SSH_CRED_ID = 'github-ssh-key'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }
                
        stage('Export HTML Static') {
            steps {
                sh "docker run --rm -v \${WORKSPACE}:/app -w /app -e NEXT_PUBLIC_BASE_PATH=/${APP_NAME} ${APP_NAME}:${BUILD_NUMBER} npm run build"
            }
        }
        
        stage('Deploy GitHub Pages') {
            steps {
                sshagent(credentials: [env.SSH_CRED_ID]) {
                    sh """
                        cd out
                        touch .nojekyll
                        git init
                        git checkout -b gh-pages
                        git config user.name "${GITHUB_USER}"
                        git config user.email "${GITHUB_EMAIL}"
                        git add .
                        git commit -m "Deploy build ${BUILD_NUMBER} to GitHub Pages"
                        git push -f ${GITHUB_REPO} HEAD:gh-pages
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh "docker rmi ${APP_NAME}:${BUILD_NUMBER} || true"
        }
    }
}