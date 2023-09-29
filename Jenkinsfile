pipeline {
    agent any
    tools {
        jdk "OpenJDK8"
    }

    environment {
        DOCKER_IMAGE_NAME = "luswala/train-schedule"
        PROD_DEPLOYMENT_CHOICE = "N"
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

             when {
                    anyOf {
                    branch 'master';
                    branch 'staging'
                    }
             }                   
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                                        app.inside {
                        sh 'echo Build In Progress'
                    }
                }
            }
        }
        stage('Push Docker Image') {
             when {
                    anyOf {
                    branch 'master';
                    branch 'staging'
                    }
             }                        
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-user') {
                        app.push("${env.BRANCH_NAME}_${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
         stage('CanaryDeploy') {
             when {
                 anyOf {
                    branch 'master';
                    branch 'staging'
                }
             }
             environment { 
                 CANARY_REPLICAS = 1
             }
             steps {
                sh "sed -i 's,\$DOCKER_IMAGE_NAME:\$BUILD_NUMBER,$DOCKER_IMAGE_NAME:${env.BRANCH_NAME}_${env.BUILD_NUMBER},' train-schedule-kube-canary.yml"
                sh "sed -i 's,\$CANARY_REPLICAS,$CANARY_REPLICAS,' train-schedule-kube-canary.yml"
                sh "cat train-schedule-kube-canary.yml"

                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get pods"
                 sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config  apply -f train-schedule-kube-canary.yml'

                 
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get deployments"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get svc"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get pods"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get hpa"


             }
         }        
         //Code should be tested in the staging branch before deployment
         stage('Production Deployment Confirmation') {
            steps('Input') {
                
                 def PROD_DEPLOYMENT_CHOICE = input(
                            id: 'PROD_DEPLOYMENT_CHOICE', message: 'Do you want to deploy code to the Production?',
                            parameters: [

                                    string(defaultValue: 'No',
                                            description: 'Deploy to Production',
                                            name: 'Yes'),
                                    string(defaultValue: 'No',
                                            description: 'Skip Prod Deployment',
                                            name: 'No'),
                            ])

            }
        }

        stage('DeployToProduction') {
                         when {
                    anyOf {
                    branch 'master';
                    expression{env.PROD_DEPLOYMENT_CHOICE == "YES"}
                    }
             }   
            environment { 
                CANARY_REPLICAS = 0
            }            
            steps {

                 sh "sed -i 's,\$DOCKER_IMAGE_NAME:\$BUILD_NUMBER,$DOCKER_IMAGE_NAME:${env.BRANCH_NAME}_${env.BUILD_NUMBER},' train-schedule-kube.yml"
                 sh "cat train-schedule-kube.yml"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get pods"
                 sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config  apply -f train-schedule-kube.yml'

                 
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get deployments"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get svc"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get pods"
                 sh "kubectl --kubeconfig=/var/lib/jenkins/.kube/config get hpa"
            }


            
        }
    }
}