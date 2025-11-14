pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/PuneethReddyHC/online-shopping-system-advanced.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'sudo apt update'
                sh 'sudo apt install -y php php-mysql apache2 unzip'
            }
        }

        stage('Deploy Code') {
            steps {
                sh '''
                    sudo rm -rf /var/www/html/shop2
                    sudo mkdir -p /var/www/html/shop2
                    sudo cp -r * /var/www/html/shop2/
                '''
            }
        }

        stage('Apache VirtualHost') {
            steps {
                sh '''
                    echo "<VirtualHost *:8081>
                        DocumentRoot /var/www/html/shop2
                        <Directory /var/www/html/shop2>
                            AllowOverride All
                            Require all granted
                        </Directory>
                    </VirtualHost>" | sudo tee /etc/apache2/sites-available/shop2.conf

                    sudo a2ensite shop2.conf
                    sudo a2enmod rewrite
                    sudo systemctl restart apache2
                '''
            }
        }
    }

    post {
        success {
            echo "Website deployed on PORT 8081"
        }
    }
}
