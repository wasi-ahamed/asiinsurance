pipeline {
    agent any

    environment {
        PATH = "/usr/bin:$PATH"
        tag = "1.0"
        dockerHubUser = "wasiahamed"
        containerName = "insure-me"
        httpPort = "8081"
    }

    stages {
        stage("Code Clone") {
            steps {
                git branch: 'master', url: 'https://github.com/wasi-ahamed/asiinsurance.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t ${dockerHubUser}/${containerName}:${tag} ."
            }
        }

        stage("Push Image to DockerHub") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerHubAccount',
                        passwordVariable: 'dockerPassword',
                        usernameVariable: 'dockerUser'
                    )
                ]) {
                    sh """
                        docker login -u $dockerUser -p $dockerPassword
                        docker push $dockerUser/$containerName:$tag
                    """
                }
            }
        }

        stage("Docker Container Deployment") {
            steps {
                sh """
                    docker rm -f $containerName || true
                    docker pull $dockerHubUser/$containerName:$tag
                    docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag
                """
                echo "Application started on port: ${httpPort} (http)"
            }
        }
    }
}
