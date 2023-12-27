pipeline {
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_TOKEN_JENKINS')
    }
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage("Compile") {
            steps {
                sh "./gradlew compileJava"
            }
        }
        stage("Build") {
           steps {
               sh "./gradlew build"
               sh "cp ./build/libs/MiniBoard-0.0.1-SNAPSHOT.jar ./docker/smboard/"
           } 
        }
        stage("Docker Login") {
           steps {
               sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin" 
           }
        }
        stage("Docker Image Build") {
           steps {
               sh "docker build -t redleon1/apache2_smboard:${BUILD_NUMBER}" ./docker/apache2/"
               sh "docker build -t redleon1/smboard_smboard:${BUILD_NUMBER}" ./docker/smboard/"
               sh "docker build -t redleon1/mariadb_smboard:${BUILD_NUMBER}" ./docker/mariadb/"
           }
        }
        stage("Docker Image Push") {
           steps {
               sh "docker push redleon1/apache2_smboard:${BUILD_NUMBER}"
               sh "docker push redleon1/smboard_smboard:${BUILD_NUMBER}" 
               sh "docker push redleon1/mariadb_smboard:${BUILD_NUMBER}" 
           } 
        }
        stage("Docker Image Clean up") {
           steps {
               sh "docker image rm redleon1/apache2_smboard:${BUILD_NUMBER}" 
               sh "docker image rm redleon1/smboard_smboard:${BUILD_NUMBER}" 
               sh "docker image rm redleon1/mariadb_smboard:${BUILD_NUMBER}" 
           }
        }
        stage("Deploy") {
           steps {
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/apache2.yml"
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/smboard.yml"
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/mariadb.yml"
               sh "kubectl apply -f ./kubernetes/mariadb.yml"
               sh "kubectl apply -f ./kubernetes/smboard.yml"
               sh "kubectl apply -f ./kubernetes/apache2.yml"
               sh "kubectl apply -f ./kubernetes/ingress.yml"
           } 
        }
        post {
                success {
                    slackSend (
                        channel: "#it교육",
                        color: "#2C953C",
                        message: "smboard 배포가 성공하였습니다."
                    )
                    echo "Completed Server Deploy"
                }
                failure {
                    slackSend (
                        channel: "#it교육",
                        color: "#FF3232",
                        message: "smboard 배포가 실패하였습니다.\n" +
                    )
                    echo "Fail Server Deploy"
                }
        }
    }
}
