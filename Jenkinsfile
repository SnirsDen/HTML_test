#!/usr/bin/env groovy

pipeline {
    agent any

    stages {
        stage('Клонирование репозитория') {
            steps {
                // Клонирование вашего репозитория
                sh 'git clone https://github.com/SnirsDen/HTML_test.git'
            }
        }

        stage('Сборка и запуск Docker контейнера') {
            steps {
                script {
                // Запуск контейнера с монтированием директории
                sh 'docker run -d -p 80:80 -v ./HTML_test:/usr/share/nginx/html --name nginx-container nginx:latest'
                }
            }
        }

        stage('Проверка') {
            steps {
                // Выполнение curl для проверки работоспособности
                sh 'curl http://localhost'
            }
        }
        
    
        stage('Получение списка IP-адресов') {
            steps {
                script {
                    sh 'gcloud config set project my-diplom-472008 --quiet'
                    // Получаем список внешних IP-адресов для машин с именами prod* и dev*
                    def remoteIPs = sh(script: '''
                        gcloud compute instances list --filter="name~'^prod.*' OR name~'^dev.*'" --format="value(networkInterfaces[0].accessConfigs[0].natIP)" --quiet
                    ''', returnStdout: true).trim()
        
                    // Разделяем результат на отдельные IP-адреса
                    env.REMOTE_IPS = remoteIPs.split("\\s+")
                    println "IP Addresses: ${env.REMOTE_IPS}"
                }
            }
        }

        stage('Копирование файлов на удаленные сервера') {
            steps {
                script {
                    sh 'gcloud config set project my-diplom-472008 --quiet'
                    // Получение списка IP-адресов для машин с именами prod* и dev*
                    def remoteIPs = sh(script: '''
                        gcloud compute instances list --filter="name~'^prod.*' OR name~'^dev.*'" --format="value(networkInterfaces[0].accessConfigs[0].natIP)" --quiet
                    ''', returnStdout: true).trim()
            
                 // Разделяем результат по пробельным символам и обрабатываем каждый IP
                    remoteIPs.split('\\s+').each { ip ->
                        // Создаем временную директорию на удаленном сервере
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'mkdir -p /tmp/HTML_test'"                
                        // Копируем файлы на удаленный сервер
                        sh "scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem -r ./pet-website/* ubuntu@${ip}:/tmp/HTML_test/"                
                        // Синхронизируем файлы в веб-директорию (с сохранением прав)
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'sudo rsync -av /tmp/HTML_test/ /var/www/html/'"               
                        // Удаляем временную директорию
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'rm -rf /tmp/HTML_test'"
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'pwd'
            // Остановка и удаление контейнера после завершения пайплайна
            sh 'docker stop nginx-container || true'
            sh 'docker rm nginx-container || true'
            deleteDir()
            script {
            def message = "Сборка завершена. Результат: ${currentBuild.result}"
//            def chatId = "" // Укажите ID чата, куда хотите отправить сообщение
//            def token = "6178888383:AAGob2W61BfTrM0buDelKt5dL2T53z0VAOQ" // Ваш токен бота
//            sh "curl -s -X POST https://api.telegram.org/bot${token}/sendMessage -d chat_id=${chatId} -d text='${message}'"
        }
        }
    }
}
