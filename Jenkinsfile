@Library("shared") _
pipeline {
    agent { label "red" }
    stages {
        stage("Hello"){
            steps{
                script{
                    hello()
                }
            }
        }
        stage("Code") {
            steps {
                script{
                    clone ("https://github.com/hrushikesh0808/django-notes-app.git","main")
                }
            }
        }
        stage("Build") {
            steps {
                script{
                    docker_build("hrushikeshdhol21", "notes-app", "latest")
                } 
            }
        }
        stage("Push to DockerHub") {
            steps {
                script{
                    docker_push("notes-app","latest","hrushikeshdhol21")
                }
            }
        }
        stage("Deploy") {
            steps {
                echo "Deploying the code to the server"
                sh "docker compose down && docker compose up -d"
            }
        }
    }
}
