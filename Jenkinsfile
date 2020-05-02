#! #!/bin/groovy

//Pipeline for Gradle build to be used on minikube

pipeline {
    agent any
    stages {
        stage('Checkout Code from GitHub') {
            steps {
                echo 'Running Build Automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainschedule.zip'
            }

        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("prashantvyas/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("$env.BUILD_NUMBER")
                        app.push("latest")
                    }
                }
                }
            }
        }
        stage('Deploy To minikube') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy To minikube?'
                milestone(1) {
                    script {
                        env.KUBE_CONTEXT = 'minikube'
                        env.NAMESPACE = 'test-dev'
                        try {
                            stage("Minikube: DEV Deploy") {
                            withKubeConfig([credentialsId: 'kube-config', variable: 'KUBECONFIG']) {
                                script {
                                    env.KUBE_CONTEXT = 'minikube'
                                    env.NAMESPACE = 'test-dev'
                                }
                                try {
                                    sh """
                                        export KUBE_CONTEXT=${KUBE_CONTEXT}
                                        export NAMESPACE=${NAMESPACE}
                                    """
                                } catch (error) {
                                    sh """
                                      kubectl config use-context ${KUBE_CONTEXT} 
                                      kubectl run train-schedule -n ${NAMESPACE} --image=prashantvyas/train-schedule:latest
                                    """
                                    throw error
                                }
                            }
                            }
                        } catch (error) {
                            throw error
                        }
                    }
                }

            }
        }
    }
    }