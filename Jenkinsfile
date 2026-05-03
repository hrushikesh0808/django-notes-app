@Library("shared") _
pipeline {
    agent { label "red" }
    stages {
        stage("Hello") {
            steps {
                script {
                    hello()
                }
            }
        }
        stage("Code") {
            steps {
                script {
                    clone("https://github.com/hrushikesh0808/django-notes-app.git", "main")
                }
            }
        }
        stage("Build") {
            steps {
                script {
                    docker_build("hrushikeshdhol21", "notes-app", "latest")
                }
            }
        }
        stage("Push to DockerHub") {
            steps {
                script {
                    docker_push("notes-app", "latest", "hrushikeshdhol21")
                }
            }
        }
        stage("Backup") {
            steps {
                echo "Taking backup of current deployment"
                sh """
                    BACKUP_DIR=/home/webadmin/backups/\$(date +%Y%m%d_%H%M%S)
                    mkdir -p \$BACKUP_DIR
                    cp /home/webadmin/workspace/DjangoCICD/docker-compose.yml \$BACKUP_DIR/ 2>/dev/null || true
                    cp /home/webadmin/workspace/DjangoCICD/.env \$BACKUP_DIR/ 2>/dev/null || true
                    docker logs django_cont > \$BACKUP_DIR/django_cont.log 2>&1 || true
                    docker logs nginx_cont  > \$BACKUP_DIR/nginx_cont.log  2>&1 || true
                    docker logs db_cont     > \$BACKUP_DIR/db_cont.log     2>&1 || true
                    docker tag django_app:latest django_app:backup_\$(date +%Y%m%d_%H%M%S) 2>/dev/null || true
                    echo "Backup saved to \$BACKUP_DIR"
                    ls -dt /home/webadmin/backups/*/ | tail -n +6 | xargs rm -rf 2>/dev/null || true
                """
            }
        }
        stage("Deploy") {
            steps {
                echo "Deploying the code to the server"
                sh '''
                    # Write correct .env (quoted EOF prevents variable expansion)
                    cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=test_db
DB_NAME=test_db
DB_USER=root
DB_PASSWORD=root
DB_PORT=3306
DB_HOST=db
DEBUG=0
SECRET_KEY=django-insecure-replace-this-in-production
ALLOWED_HOSTS=*
EOF

                    docker compose down
                    docker compose up -d --build

                    echo "Waiting for containers to become healthy..."
                    sleep 30

                    # Show final status
                    docker compose ps
                '''
            }
        }
    }
    post {
        success {
            echo "Deployment successful!"
            // Use docker ps directly — no dependency on working directory
            sh "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
        }
        failure {
            echo "Deployment FAILED — check logs"
            sh "docker ps -a --format 'table {{.Names}}\t{{.Status}}' || true"
            sh "docker logs django_cont --tail=30 || true"
        }
    }
}
