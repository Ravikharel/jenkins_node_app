pipeline { 
    agent any
    environment {
        HARBOR_URL = "harbor.registry.local/jenkins/"
        IMAGE_NAME_FRONTEND = "frontend"
        IMAGE_NAME_BACKEND = "backend"
        IMAGE_NAME_MYSQL = "mysql"
        IMAGE_TAG = "v1"
        HARBOR_PASSWORD = "Harbor12345"
    }

    stages { 
        stage('Building the Images') {
            steps { 
                script { 
                    sh '''
                        docker image build -t ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG} -f frontend/Dockerfile frontend/
                        docker image build -t ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG} -f backend/Dockerfile backend/
                    '''
                }
            }
        }

        stage('Pushing the Images to Harbor Registry') { 
            steps { 
                script { 
                    sh '''
                        echo ${HARBOR_PASSWORD} | docker login ${HARBOR_URL} -u admin --password-stdin
                        docker push ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        docker push ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Pushing Docker Compose File to Agent Node') { 
            steps { 
                script { 
                    sh "scp docker-compose.yml vagrant@192.168.56.13:/home/vagrant/jenkins/vagrant/"
                }
            }
        }
        stage('Add CA Certificate to Agent Node') {

            agent { label 'vagrant-slave' }
            steps {
                script {
                   
                    sh 'scp /path/to/your/ca.crt vagrant@192.168.56.13:/tmp/ca.crt'

                    
                    sh '''
                        sudo cp /tmp/ca.crt /usr/local/share/ca-certificates/ca.crt
                        sudo update-ca-certificates
                    '''

                    
                    sh '''
                        sudo mkdir -p /etc/docker/certs.d/harbor.registry.local
                        sudo cp /tmp/ca.crt /etc/docker/certs.d/harbor.registry.local/ca.crt
                        sudo systemctl restart docker
                    '''
        }
    }
}

        stage('Pulling Images and Running MySQL Container on Agent Node') { 
            agent { 
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh '''
                        echo ${HARBOR_PASSWORD} | docker login ${HARBOR_URL} -u admin --password-stdin
                        docker pull ${HARBOR_URL}${IMAGE_NAME_FRONTEND}:${IMAGE_TAG}
                        docker pull ${HARBOR_URL}${IMAGE_NAME_BACKEND}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Running Docker Compose and Configuring MySQL') { 
            agent {
                label "vagrant-slave"
            }
            steps { 
                script { 
                    sh """
                        docker-compose -f /home/vagrant/jenkins/vagrant/docker-compose.yml up -d
                    """
                }
                script { 
                    sh '''
                    docker-compose exec mysql mysql -u root -proot -e "USE myapp; CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL);"
                    '''
                }
            }
        }
    }
}
