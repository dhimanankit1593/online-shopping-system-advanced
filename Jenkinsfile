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
                    apt update -y
                    apt install -y php php-mysql apache2 unzip
                '''
            }
        }

        stage('Deploy Code') {
            steps {
                sh '''
                    rm -rf /var/www/html/shop2
                    mkdir -p /var/www/html/shop2
                    cp -r * /var/www/html/shop2/
                '''
            }
        }

        stage('Apache VirtualHost') {
            steps {
                sh '''
                    echo "Listen 8081" >> /etc/apache2/ports.conf || true

                    cat <<EOF > /etc/apache2/sites-available/shop2.conf
<VirtualHost *:8081>
    DocumentRoot /var/www/html/shop2
    <Directory /var/www/html/shop2>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

                    a2ensite shop2.conf || true
                    a2enmod rewrite || true
                    systemctl restart apache2
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
