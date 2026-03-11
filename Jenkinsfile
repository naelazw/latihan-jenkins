pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build & Install') {
            steps {
                script {
                    // Menggunakan image PHP 8.2 yang stabil
                    docker.image('php:8.2-cli').inside('-u root') {
                        sh '''
                            apt-get update && apt-get install -y zip unzip
                            curl -sS [https://getcomposer.org/installer](https://getcomposer.org/installer) | php -- --install-dir=/usr/local/bin --filename=composer
                            
                            composer install --no-interaction --prefer-dist
                            cp .env.example .env
                            php artisan key:generate
                        '''
                    }
                }
            }
        }

        stage('Simulasi Test') {
            steps {
                echo " berhasil build otomatis"
            }
        }
    }
}