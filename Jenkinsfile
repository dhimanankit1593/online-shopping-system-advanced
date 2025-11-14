pipeline {
    agent any

stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/dhimanankit1593/online-shopping-system-advanced.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    sudo apt update -y
                    sudo apt install -y php php-mysql apache2 unzip
                '''
            }
        }

        stage('Deploy Code') {
            steps {
                sh '''
                    sudo rm -rf /var/www/html/shop2
                    sudo mkdir -p /var/www/html/shop2
                    sudo cp -r * /var/www/html/shop2/
                    sudo chmod -R 777 /var/www/html/shop2
                '''
            }
        }

        stage('Apache VirtualHost') {
            steps {
                sh '''
                    # Add port 8081 listener if not already present
                    if ! grep -q "Listen 8081" /etc/apache2/ports.conf; then
                        echo "Listen 8081" | sudo tee -a /etc/apache2/ports.conf
                    fi

                    # Create VirtualHost file
                    sudo bash -c 'cat <<EOF > /etc/apache2/sites-available/shop2.conf
<VirtualHost *:8081>
    DocumentRoot /var/www/html/shop2
    <Directory /var/www/html/shop2>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF'

                    sudo a2ensite shop2.conf || true
                    sudo a2enmod rewrite || true
                    sudo systemctl restart apache2
                '''
            }
        }
    }

    post {
        success {
            echo "Website deployed successfully!"
            echo "URL: http://YOUR_SERVER_IP:8081"
        }
    }
}
