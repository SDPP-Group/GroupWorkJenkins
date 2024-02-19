pipeline{
    agent any

    environment {
        IMAGE_NAME = "registry.gitlab.com/autyauth1/softdevcicd"
    }

    stages{
        // stage("Set Enviroment") {
        //     agent {
        //         label 'testNode'
        //     }
        //     steps {
                
        //         sh 'python3 -m venv myenv'
        //     }
        // }
        // stage("Install lib") {
        //     agent {
        //         label 'testNode'
        //     }
        //     steps {
        //         sh '''#!/bin/bash
        //         source myenv/bin/activate && pip install -r ./requirements.txt
                
        //         '''
        //     }
        // }
        stage("Run Unit Test") {
            agent{
                label 'testNode'
            }
            steps {
                sh 'python3 ./test_restapi.py'
            }
        }
        stage("Create Images") {
            agent {
                label 'testNode'
            }
            steps {
                sh "docker compose build"
                sh "docker ps"
            }
        }
        stage("Create Container") {
            agent{
                label 'testNode'
            }
            steps {
                sh "docker compose up -d"
            }
        }
        stage("Clone Robot") {
            agent{
                label 'testNode'
            }
            steps {
                sh "rm -rf ./robottestapi"
                withCredentials([gitUsernamePassword(credentialsId: '86130d73-f735-4fd7-b9c7-6922adffce72', gitToolName: 'git-tool')]) {
                    // Use withCredentials block to securely access credentials
                    sh 'git clone https://gitlab.com/autyauth1/robottestapi.git'
                }
            }
        }
        stage("Run Robot Test") {
            agent{
                label 'testNode'
            }
            steps {
                // sh "cd ./robottestapi && robot ./plus.robot"
                sh "cd ./robottestapi && python3 -m robot ./plus.robot"
                
            }
        }
        stage("Push Image") {
            agent{
                label 'testNode'
            }
            steps{
                    // push the image to the gitlab registry with credentials
                    withCredentials([usernamePassword(credentialsId: '86130d73-f735-4fd7-b9c7-6922adffce72', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD} registry.gitlab.com'
                        sh 'docker push registry.gitlab.com/autyauth1/softdevcicd'
                    }
                    sh 'docker rmi -f registry.gitlab.com/autyauth1/softdevcicd:latest'
                    // sh "docker push registry.gitlab.com/autyauth1/softdevcicd"
            }
        }
        stage('Clean Workspace') {
            agent {
                label 'testNode'
            }
            steps {
                sh 'docker compose -f ./compose.yml down'          
                sh 'docker system prune -a -f'
            }
        }
        stage('Pull Image from Registry') {
            agent {
                label 'pre-prodNode'
            }
            steps {
                    withCredentials([usernamePassword(credentialsId: '86130d73-f735-4fd7-b9c7-6922adffce72', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD} registry.gitlab.com'
                        sh 'docker pull registry.gitlab.com/autyauth1/softdevcicd'
                    }

            }
        }
        stage('Create Container from Image') {
            agent {
                label 'pre-prodNode'
            }
            steps {
                sh "docker compose -f ./compose.yml up -d"
            }
        }
    }
    
}