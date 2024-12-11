pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        PR_ID = "${env.CHANGE_ID ?: 'local'}"
        APP_DIR = "/var/www/preview/pr-${env.CHANGE_ID ?: 'local'}"
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Kiểm tra nếu là PR, sẽ checkout branch của PR đó
                    if (env.CHANGE_ID) {
                        git(
                            url: 'https://github.com/BenBen2109/test-jenkins.git',
                            branch: "refs/pull/${env.CHANGE_ID}/head",
                            credentialsId: 'github-token'
                        )
                    } else {
                        echo "No Pull Request detected, skipping build."
                    }
                }
            }
        }
        stage('Print Commit SHA') {
            steps {
                script {
                    env.COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Current Commit SHA: ${env.COMMIT_SHA}"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh "sudo mkdir -p ${APP_DIR}"
                sh "sudo cp ./index.html ${APP_DIR}/"
                sh "nginx -v"
                sh "sudo mkdir -p /etc/nginx/sites-available"
                sh "sudo chown -R jenkins:jenkins /etc/nginx/sites-available"
                sh """
                sudo bash -c 'cat <<EOF > /etc/nginx/sites-available/pr-local.conf
server {
    listen 80;
    server_name pr-local.example.com;
    root /var/www/preview/pr-local;
    index index.html;
    location / {
        try_files \$uri /index.html;
    }
}
EOF'
                """
                sh "sudo ln -sf /etc/nginx/sites-available/pr-local.conf /etc/nginx/sites-enabled/"
                sh "sudo systemctl reload nginx"
            }
        }
    }
    post {
        success {
            script {
                githubNotify(
                    context: 'PR Preview',
                    description: "Preview deployed at http://pr-${PR_ID}.example.com",
                    status: 'SUCCESS',
                    credentialsId: 'github-token',
                    repo: 'test-jenkins',
                    sha: "${env.COMMIT_SHA}",
                    account: 'BenBen2109'
                )
            }
        }
        failure {
            script {
                githubNotify(
                    context: 'PR Preview',
                    description: "Failed to deploy preview",
                    status: 'FAILURE',
                    credentialsId: 'github-token',
                    repo: 'test-jenkins',
                    sha: "${env.COMMIT_SHA}",
                    account: 'BenBen2109'
                )
            }
        }
    }
}