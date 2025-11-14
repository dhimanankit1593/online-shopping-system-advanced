pipeline {
    agent any

    environment {
        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "YOUR_SERVER_PUBLIC_IP"
        DEPLOY_PATH = "/var/www/html/online-shopping"
        GIT_URL = "https://github.com/PuneethReddyHC/online-shopping-system-advanced.git"
        PORT = "8081"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Pulling code from GitHub..."
                git branch: 'master', url: "${GIT_URL}"
            }
        }

        stage('Install PHP Dependencies') {
            steps {
                echo "Installing dependencies (if any)..."
                sh """
                    if [ -f composer.json ]; then
                        composer install || true
                    else
                        echo "No composer.json found, skipping composer install."
                    fi
                """
            }
        }

        stage('Deploy to Server') {
            steps {
                echo "Deploying files directly to server..."
                sh """
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                        sudo mkdir -p ${DEPLOY_PATH}
                    '

                    rsync -avz --delete \
                        ${WORKSPACE}/ \
                        ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}
                """
            }
        }

        stage('Apache VirtualHost Setup') {
            steps {
                echo "Configuring Apache for port ${PORT}..."
                sh """
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '

                        # Add port if not exists
                        sudo grep -qxF "Listen ${PORT}" /etc/apache2/ports.conf || echo "Listen ${PORT}" | sudo tee -a /etc/apache2/ports.conf

                        # Create virtual host
                        sudo bash -c "cat > /etc/apache2/sites-available/online-shopping.conf" << EOF
<VirtualHost *:${PORT}>
    DocumentRoot ${DEPLOY_PATH}
    <Directory ${DEPLOY_PATH}>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

                        sudo a2ensite online-shopping.conf
                    '
                """
            }
        }

        stage('Restart Apache') {
            steps {
                sh """
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'sudo systemctl restart apache2'
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "curl -Is http://${DEPLOY_HOST}:${PORT} | head -n 5"
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Website deployed successfully!"
        }
        failure {
            echo "FAILED: Please check pipeline logs."
        }
    }
}
