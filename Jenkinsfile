pipeline {
    agent { label "dev" }
    stages{
        stage("code"){
            steps{
                git url: "https://github.com/devashish-pandey/2-tier-flask-app.git", branch: "master"
            }
        }
        stage("Build"){
            steps{
                sh "docker build -t two-tier-flask-app ."
            }
        }
        stage("test"){
            steps{
                echo "test"
            }
        }
        stage("Push"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: "dockerCred",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                    )]){
                        sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                        sh "docker image tag two-tier-flask-app ${env.dockerHubUser}/two-tier-flask-app"
                        sh "docker push ${env.dockerHubUser}/two-tier-flask-app:latest"
                    }
                }
            }
        stage("Deploy"){
            steps{
                sh "docker compose up -d --build"
            }
        }
    }
}
