pipeline {
    agent any 
    
    tools {
        dockerTool 'docker'
    }
    
    // Se déclenche automatiquement sur un push GitHub
    triggers { githubPush() } 

    environment {
        // URL SSH du dépôt
        GITHUB_REPO = 'git@github.com:valentin-provost/pipeline-as-code.git'
        // Nom EXACT du dépôt
        APP_NAME = 'pipeline-as-code'
        // Labels pour les commits de déploiement
        GITHUB_USER = 'Jenkins CI'
        GITHUB_EMAIL = 'jenkins@ci.local'
        // L'ID du credential SSH que l'on vient de créer
        SSH_CRED_ID = 'github-ssh-key'
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupère le code depuis GitHub
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                // Construit l'image Docker
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }
        
        stage('Install') {
            steps {
                // Installe les dépendances avec npm ci dans le container
                sh "docker run --rm -v \${WORKSPACE}:/app -w /app ${APP_NAME}:${BUILD_NUMBER} npm ci"
            }
        }
        
        stage('Export HTML Static') {
            steps {
                // Génère le dossier out/ contenant le site web statique
                sh "docker run --rm -v \${WORKSPACE}:/app -w /app -e NEXT_PUBLIC_BASE_PATH=/${APP_NAME} ${APP_NAME}:${BUILD_NUMBER} npm run build"
            }
        }
        
        stage('Deploy GitHub Pages') {
            steps {
                // Utilise la clé SSH pour pousser le résultat sur GitHub
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
            // Nettoie l'image Docker après l'exécution pour ne pas saturer le serveur
            sh "docker rmi ${APP_NAME}:${BUILD_NUMBER} || true"
        }
    }
}