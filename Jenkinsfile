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
                        # 1. Buat folder tujuan di server target
                        ssh -o StrictHostKeyChecking=no root@172.27.12.115 "mkdir -p /root/prod_server"
                        
                        # 2. Kirim file hasil build ke server target
                        scp -o StrictHostKeyChecking=no -r ./* root@172.27.12.115:/root/prod_server/
                        
                        # 3. Jalankan Laravel di dalam container Docker di server target
                        ssh -o StrictHostKeyChecking=no root@172.27.12.115 "
                            cd /root/prod_server
                            # Hapus container lama jika ada agar port tidak bentrok
                            docker rm -f laravel-online || true
                            
                            # Jalankan container baru di port 8001
                            docker run -d --name laravel-online \
                                -p 8001:8000 \
                                -v \$(pwd):/var/www/html \
                                -w /var/www/html \
                                php:8.4-cli php artisan serve --host=0.0.0.0
                        "
                        echo "ALHAMDULILLAH! Deploy Berhasil dan Sudah Running di Port 8001"
                    '''
                }
            }
        }
    } 
} 