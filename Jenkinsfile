pipeline {
    agent { label 'slave-node' }
    
    environment {
        GITHUB_REPO_URL = "https://github.com/M95kandan/end-to-end-CI-CD"
        DOCKERHUB_USERNAME = "m95kandan"
        DOCKERHUB_PASSWORD = "M#95kandan"
        DOCKERHUB_REPOSITORY = "m95kandan/website"
        BUILD_NUMBER_TAG = "${BUILD_NUMBER}"
        RETRY_COUNT = 3
        RETRY_INTERVAL = 60  // seconds
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the repository
                    git branch: 'main', url: 'https://github.com/M95kandan/end-to-end-CI-CD'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    retry(RETRY_COUNT) {
                        ansiblePlaybook(
                            playbook: 'docker-build.yml',
                            inventory: 'inventory',
                            extraVars: [
                                github_repo_url: GITHUB_REPO_URL,
                                dockerhub_username: DOCKERHUB_USERNAME,
                                dockerhub_password: DOCKERHUB_PASSWORD,
                                dockerhub_repository: DOCKERHUB_REPOSITORY,
                                build_number_tag: BUILD_NUMBER_TAG
                            ]
                        )
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    retry(RETRY_COUNT) {
                        ansiblePlaybook(
                            playbook: 'docker-push.yml',
                            inventory: 'inventory',
                            extraVars: [
                                dockerhub_username: DOCKERHUB_USERNAME,
                                dockerhub_password: DOCKERHUB_PASSWORD,
                                dockerhub_repository: DOCKERHUB_REPOSITORY,
                                build_number_tag: BUILD_NUMBER_TAG
                            ]
                        )
                    }
                }
            }
        }

        stage('Approved') {
            steps {
                script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput

                    if(userInput == true) {
                        // do action
                    } else {
                        // not do action
                        echo "Action was aborted."
                    }
                }
            }
        }

        stage('Run K8sMasterSlave playbook') {
            steps {
                script {
                    retry(RETRY_COUNT) {
                        ansiblePlaybook(
                            playbook: 'K8sMasterSlave.yml',
                            inventory: 'inventory',
                           
                        )
                    }
                }
            }
        }

        stage('Deploy Application on Kubernetes') {
            steps {
                script {
                    retry(RETRY_COUNT) {
                        ansiblePlaybook(
                            playbook: 'k8sAppDeploy.yml',
                            inventory: 'inventory',
                           
                        )
                    }
                }
            }
        }
    }
}
