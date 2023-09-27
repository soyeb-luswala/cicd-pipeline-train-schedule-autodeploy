pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "luswala/train-schedule"
    }
    stages {
        stage("Clone GIt"){
            steps{
            git url: 'https://github.com/soyeb-luswala/cicd-pipeline-train-schedule-autodeploy'
            }
        }
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                                        app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
                        when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        // stage('CanaryDeploy') {
        //     when {
        //         branch 'master'
        //     }
        //     environment { 
        //         CANARY_REPLICAS = 1
        //     }
        //     steps {
        //         kubernetesDeploy(
        //             kubeconfigId: 'kubeconfig',
        //             configs: 'train-schedule-kube-canary.yml',
        //             enableConfigSubstitution: true
        //         )
        //     }
        // }        
        stage('DeployToProduction') {
                        when {
                branch 'master'
            }
            // environment { 
            //     CANARY_REPLICAS = 0
            // }            
            steps {

                 sh "kubectl --kubeconfig=/home/devops_user/mykubeconfig/config get pods"
                 sh 'kubectl --kubeconfig=/home/devops_user/mykubeconfig apply -f deployment.yaml'
                 sh 'kubectl --kubeconfig=/home/devops_user/mykubeconfig apply -f app-service.yaml'
                 
                 sh "kubectl --kubeconfig=/home/devops_user/mykubeconfig/config get deployments"
                 sh "kubectl --kubeconfig=/home/devops_user/mykubeconfig/config get svc"
                 sh "kubectl --kubeconfig=/home/devops_user/mykubeconfig/config get pods"
            }


            
        }
    }
}