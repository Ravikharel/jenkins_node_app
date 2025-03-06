pipeline{ 
    agent any
    environment{
        HARBOR_URL = "harbor.registry.local/jenkins/"
        IMAGE_NAME = frontend
        IMAGE_NAME1 = backend
        IMAGE_NAME2 = mysql 
        IMAGE_TAG = v1
    }
    stages{ 
        stage('building the images'){
            steps{ 
                script{ 
                    sh '''
                        docker image build -t ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG} -f frontend/Dockerfile frontend/ 
                        docker image build -t ${HARBOR_URL}/${IMAGE_NAME1}:${IMAGE_TAG} -f backend/Dockerfile backend/ 
                    '''
                }
            } ~

        }
        stage('Pushing the image to the harbor.registry.local'){ 
            steps{ 
                script{ 
                    sh '''
                        echo Harbor12345 | docker login harbor.registry.local -u admin --password-stdin
                        docker push ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${HARBOR_URL}/${IMAGE_NAME1}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Pushing the docker-compose file to the agent node'){ 
            steps{ 
                script{ 
                    sh "scp docker-compose.yml vagrant@$192.168.56.13:/home/vagrant/jenkins/vagrant/"
                }
            }
        }
        stage('Pulling the image and running the mysql container in the agent node'){
            agent{ 
                label "vagrant-slave"
            }
            steps{ 
                script{ 
                    sh '''
                        echo Harbor12345 | docker login harbor.registry.local -u admin --password-stdin
                        docker pull ${HARBOR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker pull ${HARBOR_URL}/${IMAGE_NAME1}:${IMAGE_TAG}
                        
                    '''
                }
            
            }

        }
        stage('Running the docker compose file '){ 
            agent{
                label "vagrant-slave"

            }
            steps{ 
                script{ 
                    sh """
                        docker compose -f /home/vagrant/jenkins/vagrant/docker-compose.yml up -d
                    """
                }
                script{ 
                    sh '''
                    docker compose exec mysql  mysql -u root -proot -e "USE myapp; CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL);"
                    '''
                }
            }
        }
    }
}