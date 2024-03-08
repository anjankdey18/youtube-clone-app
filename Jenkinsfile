pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node 16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
         DOCKER_TAG = getVersion()
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/anjankdey18/youtube-clone-app.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=myApp-cicd \
                    -Dsonar.projectKey=myApp-cicd'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
        //  stage("Docker Build & Push"){
        //      steps{
        //          script{
        //           withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
        //               sh "docker build -t youtube-clone ."
        //               sh "docker tag youtube-clone anjandey18/youtube-clone:latest "
        //               sh "docker push anjandey18/youtube-clone:latest "
        //             }
        //         }
        //     }
        // }
        stage('Docker Build'){
            steps{
                sh "docker build . -t anjandey18/youtube-clone-app:${DOCKER_TAG} "
            }
        }
                        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u anjandey18 -p ${dockerHubPwd}"
                }
                sh "docker push anjandey18/youtube-clone-app:${DOCKER_TAG} "
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image anjandey18/youtube-clone-app:latest > trivyimage.txt" 
            }
        }
        stage('Docker Deploy'){
            steps{
                // ansiblePlaybook credentialsId: 'docker-server-access-dev', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml', vaultTmpPath: ''
                ansiblePlaybook credentialsId: 'docker-server-access-dev', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml', vaultTmpPath: ''
            }
        }

        // stage('Deploy to Kubernets'){
        //     steps{
        //         script{
        //             dir('Kubernetes') {
        //               withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //               sh 'kubectl delete --all pods'
        //               sh 'kubectl apply -f deployment.yml'
        //               sh 'kubectl apply -f service.yml'
        //               }   
        //             }
        //         }
        //     }
        // }
    }
    // post {
    //  always {
    //     emailext attachLog: true,
    //         subject: "'${currentBuild.result}'",
    //         body: "Project: ${env.JOB_NAME}<br/>" +
    //             "Build Number: ${env.BUILD_NUMBER}<br/>" +
    //             "URL: ${env.BUILD_URL}<br/>",
    //         to: 'ashfaque.s510@gmail.com',                              
    //         attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    //     }
    // }
}

def getVersion(){
    def commitHash = sh lavel: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}