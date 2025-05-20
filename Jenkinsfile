pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK21"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
        REGION = "ap-northest-2"
        AWS_CREDENTIALS_NAME = "AWSCredentials"
    }
    
    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: "https://github.com/rosescent00/spring-petclinic.git",
                    branch: 'main'
            }
            post {
                success {
                    echo 'Git Clone Success'
                }
                failure {
                    echo 'Git Clone Fail'
                }
            }
        }
        
        // Mavenbuild 작업
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package' //test에러 무시
            }
        }


        //Docker image 생성
        stage('Docker Image Build') {
            steps {
                echo 'Docke Image Build'
                dir("${env.WORKSPACE}") {
                    sh '''
                    docker build -t spring-petclinic:$BUILD_NUMBER .
                    docker tag spring-petclinic:$BUILD_NUMBER rosescent00/spring-petclinic:latest
                    '''
                }
            }
        }

        //Docker images push
        stage('Docker Image Push') {
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push rosescent00/spring-petclinic:latest
                '''
            }
        }

        // Remove Docker Image
        stage('Remove Docker Image') {
            steps {
                sh '''
                docker rmi spring-petclinic:$BUILD_NUMBER
                docker rmi rosescent00/spring-petclinic:latest
                '''

            }
        }
        
    }
}
