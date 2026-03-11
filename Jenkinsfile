pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                // Mengambil kode terbaru dari repositori GitHub
                checkout scm
            }
        }

        stage('Build & Install') {
            steps {
                script {
                    // Menggunakan container PHP 8.4 untuk proses build Laravel
                    docker.image('php:8.4-cli').inside('-u root') {
                        sh '''
                            # Update dan install tools pendukung
                            apt-get update && apt-get install -y zip unzip curl
                            
                            # Install Composer
                            curl -sS https://getcomposer.org/installer -o composer-setup.php
                            php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                            rm composer-setup.php
                            
                            # Install dependensi Laravel
                            composer install --no-interaction --prefer-dist
                            
                            # Konfigurasi file .env dan generate key
                            if [ ! -f .env ]; then cp .env.example .env; fi
                            php artisan key:generate
                        '''
                    }
                }
            }
        }

        stage('Deploy to Simulation Server') {
            steps {
                // Menggunakan SSH Agent dengan credential 'ssh-prod' yang sudah kamu buat
                sshagent(credentials: ['ssh-prod']) {
                    sh '''
                        # 1. Masuk ke WSL via SSH untuk membuat folder tujuan (Pintu Resmi)
                        ssh -o StrictHostKeyChecking=no root@172.27.12.115 "mkdir -p /root/prod_server"
                        
                        # 2. Mengirim file menggunakan SCP (Secure Copy) ke IP WSL kamu
                        # Ini akan menggunakan kunci 'ssh-prod' secara otomatis
                        scp -o StrictHostKeyChecking=no -r ./* root@172.27.12.115:/root/prod_server/
                        echo "ALHAMDULILLAH! Deploy Berhasil "
                    '''
                }
            }
        }
    }
}