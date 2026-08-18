@Library("Shared") _
pipeline {
    agent { label "dev" }
    stages{
        stage("code"){
            steps{
                script{
                    clone("https://github.com/devashish-pandey/2-tier-flask-app.git","master")
                }
            }
        }
        stage("Trivy") {
            steps {
                sh "trivy fs . -o result.json"
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
                script{
                    docker_push("dockerCred","two-tier-flask-app")
                }
            }
        }
        stage("Deploy"){
            steps{
                sh "docker compose up -d --build"
            }
        }
    }
    post {
        success {
            emailext(
                subject:"Succesful build",
                body:"Good , succsess in build",
                to: 'pdevashish587@gmail.com'
                )
        }
        failure{
            emailext(
                subject:"failed build",
                body:"Bad , failure in build",
                to: 'pdevashish587@gmail.com'
                )
        }
    }
}
