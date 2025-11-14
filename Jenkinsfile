pipeline {
    agent any

    environment {
        // Update these values
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

        stage('Prepare Workspace') {
            steps {
                echo "Cleaning old unnecessary files..."
                sh """
                    rm -rf ${WORKSPACE}/vendor
                    rm -rf ${WORKSPACE}/build
                """
            }
        }

        stage('Install PHP Dependencies') {
            steps {
                echo "Installing composer dependencies (if composer.json exists)..."
                sh """
                    if [ -f composer.json ]; then
                        composer install || true
                    else
                        echo "No composer.json found. Skipping composer install."
                    fi
                """
            }
        }

        stage('Build Artifact') {
            steps {
                echo "Creating build directory and packaging website..."
                sh """
                    mkdir -p build
                    cp -r * build/
                """
            }
        }

        stage('Deploy to Server') {
            steps {
                echo "Deploying website to remote server on port ${PORT}..."
                sh """
                    ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                        sudo mkdir -p ${DEPLOY_PATH}
                    '
                    rsync -avz --delete build/ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}
                """
            }
        }

        stage('Configure Apache VirtualHost') {
            steps {
                echo "Configuring Apache VirtualHost for port ${PORT}..."
                sh """
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                        echo "Listen ${PORT}" | sudo tee /etc/apache2/ports.conf > /dev/null

                        sudo bash -c "cat > /etc/apache2/sites-available/online-shopping.conf" << EOF
<VirtualHost *:${PORT}>
    ServerAdmin admin@example.com
    DocumentRoot ${DEPLOY_PATH}

    <Directory ${DEPLOY_PATH}>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/online-shopping-error.log
    CustomLog \${APACHE_LOG_DIR}/online-shopping-access.log combined
</VirtualHost>
EOF

                        sudo a2ensite online-shopping.conf
                    '
                """
            }
        }

        stage('Restart Apache') {
            steps {
                echo "Restarting Apache service..."
                sh """
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                        sudo systemctl restart apache2
                    '
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Validating deployment..."
                sh """
                    curl -Is http://${DEPLOY_HOST}:${PORT} | head -n 5
                """
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Website deployed successfully on port ${PORT}!"
        }
        failure {
            echo "ERROR: Deployment failed. Check logs."
        }
    }
}
