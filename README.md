# kubernetes_jenkins_project
node {
    
    stage('git checkout'){
        git branch: 'main', url: 'https://github.com/Shivendra73/kubernetes_jenkins_project.git'
    }
    stage('sending dockerfile to ansible server'){
        sshagent(['admin_server']) {
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156'
           sh 'scp /var/lib/jenkins/workspace/pipeline_demo/Dockerfile ubuntu@172.31.95.156:/home/ubuntu'
        }
        
    }
    stage('building docker image'){
        sshagent(['admin_server']) {
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156'
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156 sudo docker image build -t $JOB_NAME:v1.$BUILD_ID -f /home/ubuntu/Dockerfile .'

        }
    }
    stage("Docker Image Tagging"){
        sshagent(['admin_server']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156 cd /home/ubuntu'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156 sudo docker image tag $JOB_NAME:v1.$BUILD_ID shivendrapatel/$JOB_NAME:v1.$BUILD_ID' 
        }
            
    }
    stage('Pushing Docker images from asible serer to docker hub'){
        sshagent(['admin_server']){
        withCredentials([string(credentialsId: 'docker_access', variable: 'docker_access')]) {
            sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156 sudo docker login -u shivendrapatel -p $docker_access "
            sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.95.156 sudo docker image push shivendrapatel/$JOB_NAME:v1.$BUILD_ID "
            
    
}
        }
    }
}
