pipeline {
    agent any
    environment {
        MAVEN_ARGS=" -e clean install"
        dockerContainerName = "bookapi_${params.ENV}"
        dockerImageName = "bookapi_api_${params.ENV}"
        SPRING_PROFILES_ACTIVE  "${params.ENV}"
    }
    parameters {
        choice(name: 'ENV', choices: ['staging', 'production'], description: 'Select Enviroinment(staging,production)')
    }
    stages {
        stage('Initialize') {
            steps {
                script{
                    if(!params.ENV) {
                        params.ENV = 'staging'
                    }
                }
            }
        }
        stage('Build') {
            steps {
                withMaven(maven: 'MAVEN_HOME'){
                    sh "mvn ${MAVEN_ARGS}"
                }
            }
        }
        
        stage('clean container') {
            steps {
                sh 'docker ps -f name=${dockerContainerName} -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -f name=${dockerContainerName} -q | xargs -r docker container rm'
                sh 'docker images -q --filter=reference=${dockerImagesName} | xargs --no-run-if-empty docker rmi -f'

            }
        }
        
        stage('docker-compose start') {
            steps {
                sh """
                 SPRING_PROFILES_ACTIVE=${SPRING_PROFILES_ACTIVE} docker-compose -f docker-compose-${params.ENV} up -d --build
                 """
            }
        }
        
    }
}