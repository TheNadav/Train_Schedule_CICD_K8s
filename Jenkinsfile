pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("anothernadav/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
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
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws_credentials_id', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh '''
                        aws eks --region us-east-1 update-kubeconfig --name eks-50tDfLCM
                        sed -i 's/$DOCKER_IMAGE_NAME/anothernadav\/train-schedule/g' train-schedule-kube.yml
                        sed -i 's/$BUILD_NUMBER/'"$BUILD_NUMBER"'/g' train-schedule-kube.yml
                        kubectl apply -f train-schedule-kube.yml
                    '''
                }
            }
        }
    }
}