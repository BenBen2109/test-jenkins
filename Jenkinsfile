pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        PR_ID = "${env.CHANGE_ID ?: 'local'}"
        APP_DIR = "/var/www/preview/pr-${env.CHANGE_ID ?: 'local'}"
        PORT = "80${env.PR_ID}" 
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "FETCH_HEAD"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        userRemoteConfigs: [[
                            url: 'https://github.com/BenBen2109/test-jenkins.git',
                            credentialsId: 'github-token'
                        ]]
                    ])
                }
            }
        }
        stage('Print Commit SHA') {
            steps {
                script {
                    env.COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh "sudo mkdir -p ${APP_DIR}"
                    sh "sudo cp ./index.html ${APP_DIR}/"
                    sh "nginx -v"
                    sh "sudo mkdir -p /etc/nginx/sites-available"
                    sh "sudo chown -R jenkins:jenkins /etc/nginx/sites-available"
                    sh """
                    sudo bash -c 'cat <<EOF > /etc/nginx/sites-available/pr-local.conf
server {
    listen ${PORT};
    server_name pr-${PR_ID}.example.com;
    root /var/www/preview/pr-${PR_ID};
    index index.html;
    
    location / {
        try_files \$uri \$uri/ /index.html;
    }
}
EOF'
                    """
                    sh "sudo ln -sf /etc/nginx/sites-available/pr-local.conf /etc/nginx/sites-enabled/"
                    sh "sudo systemctl reload nginx"
                }
            }
        }
    }
    post {
        success {
            script {
                def previewUrl = "http://pr-${PR_ID}.example.com"
                echo "${previewUrl}"
                githubNotify(
                    context: 'PR Preview',
                    description: "Preview deployed at http://pr-${PR_ID}.example.com",
                    status: 'SUCCESS',
                    credentialsId: 'github-token',
                    repo: 'test-jenkins',
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
                )
            }
        }
    }
}