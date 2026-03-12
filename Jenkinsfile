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
                sshagent(credentials: ['ssh-prod']) {
                    sh '''
                        # 1. Pastikan folder tujuan ada
                        ssh -o StrictHostKeyChecking=no root@172.27.12.115 "mkdir -p /root/prod_server"
                        
                        # 2. Kirim file hasil build ke server
                        scp -o StrictHostKeyChecking=no -r ./* root@172.27.12.115:/root/prod_server/
                        
                        # 3. Eksekusi di server target (Biar otomatis tampil kayak di browser kamu tadi)
                        ssh -o StrictHostKeyChecking=no root@172.27.12.115 "
                            cd /root/prod_server
                            docker rm -f laravel-online || true
                            
                            docker run -d --name laravel-online \
                                -p 8001:8000 \
                                -v /root/prod_server:/var/www/html \
                                -w /var/www/html \
                                php:8.4-cli php artisan serve --host=0.0.0.0
                            
                            # Jalankan perintah sakti yang tadi kita coba manual
                            docker exec laravel-online cp .env.example .env || true
                            docker exec laravel-online php artisan key:generate
                            docker exec -u root laravel-online chmod -R 777 storage bootstrap/cache
                        "
                        echo "port 8001"
                    '''
                }
            }
        }
    } 
} 