pipeline{
    agent any

    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerpasswd')
	} 

    stages{
        // stage('Lint HTML'){
        //     steps {
        //         sh 'tidy -q -e *.html'
        //     }
        // }

        stage("Docker Image Build"){
            steps{
                echo "========Building Docker Image========"
                sh 'docker build . -t akshayraina/${JOB_NAME}:v1.${BUILD_ID}'
            }
        }

        stage("Saving Image to DockerHub"){
            steps{
                echo "========Pushing Docker Image========"
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push akshayraina/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker rmi akshayraina/$JOB_NAME:v1.$BUILD_ID'
            }
        }

        stage("Transferring files to Ansible Server"){
            steps{
                echo "========Transferring files to Ansible Server========"
                sshagent(['ansible_server']){
                    sh 'ssh -o StrictHostKeyChecking=no root@10.83.191.88 cd /home/ubuntu/blue-green-deployment/'
                    sh "scp /var/lib/jenkins/workspace/${JOB_NAME}/* root@10.83.191.88:/home/ubuntu/blue-green-deployment/"
                }
            }
        }

        stage("Transferring files to Kubernetes Server"){
            steps{
                echo "========Transferring files to Kubernetes Server========"
                sshagent(['kubernetes_server']){
                    sh 'ssh -o StrictHostKeyChecking=no root@172.16.14.131 cd /home/akshay/Kubernetes/blue-green'
                    sh "scp /var/lib/jenkins/workspace/${JOB_NAME}/* root@172.16.14.131:/home/akshay/Kubernetes/blue-green"
                }
            }
        }

        stage("Deploying Blue and green deployment on Kubernetes"){
            steps{
                echo "========Deploying on Kubernetes Server========"
                sshagent(['ansible_server']){
                    sh 'ssh -o StrictHostKeyChecking=no root@10.83.191.88 ansible-playbook /home/ubuntu/blue-green-deployment/blue-green-playbook.yml'
                    
                    // sh 'ssh -o StrictHostKeyChecking=no root@10.83.191.88 ansible-playbook /home/ubuntu/blue-green-deployment/green-deployment.yml'
                }
            }
        }
        stage("Create service for blue deployment"){
            steps{
                echo "========Deploying on Kubernetes Server========"
                sshagent(['ansible_server']){
                    sh 'ssh -o StrictHostKeyChecking=no root@10.83.191.88 ansible-playbook /home/ubuntu/blue-green-deployment/blue-service.yml'
                }
            }
        }
        stage("User approve to continue"){
            steps{
                input "Are you sure to redirect traffic to green"
            }
        }

        stage("Create service for green deployment"){
            steps{
                echo "========Deploying on Kubernetes Server========"
                sshagent(['ansible_server']){
                    sh 'ssh -o StrictHostKeyChecking=no root@10.83.191.88 ansible-playbook /home/ubuntu/blue-green-deployment/green-service.yml'
                }
            }
        }   
    }
}
