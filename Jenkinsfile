pipeline {
    agent any

    environment {
        IMAGE_NAME = 'pavansai2205/reactjs_portfolio:latest'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'master', url: 'https://github.com/pavansai2205/reactjs-project.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker build -t portfolio_docker-frontend .
                            docker tag portfolio_docker-frontend:latest $IMAGE_NAME
                            docker push $IMAGE_NAME
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sshagent (credentials: ['ec2-ssh-deployer']) {
                    sh '''
                        echo "Deploying Docker container using Ansible..."
                        ansible-playbook -i /var/lib/jenkins/ansible/inventory.ini /var/lib/jenkins/ansible/deploy.yml -vv
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed.'
        }
    }
}
