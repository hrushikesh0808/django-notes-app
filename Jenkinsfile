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
                    clone("https://github.com/hrushikesh0808/django-notes-app.git","main")
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
        stage("Backup") {
            steps {
                echo "Taking backup of current deployment"
                sh """
                    BACKUP_DIR=/home/webadmin/backups/\$(date +%Y%m%d_%H%M%S)
                    mkdir -p \$BACKUP_DIR

                    # Backup compose and env files
                    cp /home/webadmin/workspace/DjangoCICD/docker-compose.yml \$BACKUP_DIR/ 2>/dev/null || true
                    cp /home/webadmin/workspace/DjangoCICD/.env \$BACKUP_DIR/ 2>/dev/null || true

                    # Backup running container logs before shutdown
                    docker logs django_cont > \$BACKUP_DIR/django_cont.log 2>&1 || true
                    docker logs nginx_cont  > \$BACKUP_DIR/nginx_cont.log  2>&1 || true
                    docker logs db_cont     > \$BACKUP_DIR/db_cont.log     2>&1 || true

                    # Tag current docker image as backup
                    docker tag django_app:latest django_app:backup_\$(date +%Y%m%d_%H%M%S) 2>/dev/null || true

                    echo "Backup saved to \$BACKUP_DIR"

                    # Keep only last 5 backups to save disk space
                    ls -dt /home/webadmin/backups/*/ | tail -n +6 | xargs rm -rf 2>/dev/null || true
                """
            }
        }
        stage("Deploy") {
            steps {
                echo "Deploying the code to the server"
                sh "cp /home/webadmin/workspace/DjangoCICD/.env .env"
                sh "docker compose down && docker compose up -d --build"
            }
        }
    }
    post {
        success {
            echo "Deployment successful!"
            sh "docker compose ps"
        }
        failure {
            echo "Deployment FAILED — check logs"
            sh "docker compose logs --tail=50 || true"
        }
    }
}
