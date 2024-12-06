pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = "docker-compose.yaml"
        DOCKER_IMAGE_BACKEND = "proyecto-taller-backend"
        DOCKER_IMAGE_FRONTEND = "proyecto-taller-frontend"
        DOCKER_IMAGE_DB = "postgres:16"
        VPS_USER = "root"
        VPS_IP = "161.35.97.157"
        DEPLOY_PATH = "/home/Cachi123/proyecto-taller"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Clone Main Repository') {
            steps {
                script {
                    sh 'git clone https://github.com/Taller-Web-G4/camayopolis-docker.git .'
                }
            }
        }

        stage('Clone Repositories Inside Folders') {
            steps {
                script {
                    dir('camayopolis-backend') {
                        sh 'git clone https://github.com/Taller-Web-G4/camayopolis-backend.git .'
                    }
                    dir('camayopolis-frontend') {
                        sh 'git clone https://github.com/Taller-Web-G4/camayopolis-frontend.git .'
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_BACKEND} -f camayopolis-backend/Dockerfile camayopolis-backend"
                    sh "docker build -t ${DOCKER_IMAGE_FRONTEND} -f camayopolis-frontend/Dockerfile camayopolis-frontend"
                }
            }
        }

        stage('Deploy to VPS') {
            steps {
                script {
                    def vpsPassword = 'cAchi123?hola'

                    // Crear el archivo .env en el VPS con los valores proporcionados
                    sh """
                    sshpass -p '${vpsPassword}' ssh ${VPS_USER}@${VPS_IP} 'echo -e "POSTGRES_USER=dannyortiz\nPOSTGRES_PASSWORD=Tallerweb#1\nPOSTGRES_DB=camayopolis\nSPRING_DATASOURCE_URL=jdbc:postgresql://taller.klad3.me:5432/camayopolis\nSPRING_DATASOURCE_USERNAME=dannyortiz\nSPRING_DATASOURCE_PASSWORD=Tallerweb#1" > ${DEPLOY_PATH}/.env'
                    """

                    // Copiar los directorios y archivos al VPS
                    sh """
                    ssh-keyscan -H ${VPS_IP} >> ~/.ssh/known_hosts
                    sshpass -p '${vpsPassword}' ssh ${VPS_USER}@${VPS_IP} 'mkdir -p ${DEPLOY_PATH}'
                    sshpass -p '${vpsPassword}' scp -r camayopolis-backend ${VPS_USER}@${VPS_IP}:${DEPLOY_PATH}
                    sshpass -p '${vpsPassword}' scp -r camayopolis-frontend ${VPS_USER}@${VPS_IP}:${DEPLOY_PATH}
                    sshpass -p '${vpsPassword}' scp ${DOCKER_COMPOSE_FILE} ${VPS_USER}@${VPS_IP}:${DEPLOY_PATH}
                    """

                    // Asegurarse de tener Docker Compose instalado y levantar los contenedores
                    sh """
                    sshpass -p '${vpsPassword}' ssh ${VPS_USER}@${VPS_IP} 'sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-\$(uname -s)-\$(uname -m)" -o /usr/local/bin/docker-compose'
                    sshpass -p '${vpsPassword}' ssh ${VPS_USER}@${VPS_IP} 'sudo chmod +x /usr/local/bin/docker-compose'
                    sshpass -p '${vpsPassword}' ssh ${VPS_USER}@${VPS_IP} 'cd ${DEPLOY_PATH} && docker compose down && docker compose pull && docker compose up -d --build'
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado.'
        }
    }
}