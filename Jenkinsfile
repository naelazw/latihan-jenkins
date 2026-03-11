pipeline { agent any
    stages {
        stage('Checkout SCM') {
            steps {
                // Mengambil kode dari repositori GitHub
                checkout scm
            }
        }

        stage('Build & Install') {
            steps {
                script {
                    // Menggunakan image PHP 8.2 yang stabil
                    docker.image('php:8.4-cli').inside('-u root') {
                        sh '''
                            # 1. Update dan install tools dasar
                            apt-get update
                            apt-get install -y zip unzip curl

                            # 2. Install Composer (Cara yang lebih bersih dan aman)
                            curl -sS https://getcomposer.org/installer -o composer-setup.php
                            php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                            rm composer-setup.php

                            # 3. Install dependensi Laravel
                            composer install --no-interaction --prefer-dist

                            # 4. Setup environment file
                            if [ ! -f .env ]; then
                                cp .env.example .env
                            fi

                            # 5. Generate Application Key
                            php artisan key:generate
                        '''
                    }
                }
            }
        }

        stage('Simulasi Test') {
            steps {
                echo "ALHAMDULILLAH! Berhasil build otomatis"
            }
        }
    }
}