pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-admin')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t ngotheson/springboot .'
                    sh 'docker push ngotheson/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create sonnt-dev || echo "this network exists"'
                sh 'docker container stop sonnt-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm sonnt-mysql-data || echo "no volume"'

                sh "docker run --name sonnt-mysql --rm --network mysql-dev -v sonnt-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_LOGIN_PSW -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i sonnt-mysql mysql --user=root --password=$MYSQL_ROOT_LOGIN_PSW < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull ngotheson/springboot'
                sh 'docker container stop ngotheson-springboot || echo "this container does not exist" '
                sh 'docker network create sonnt-dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name ngotheson-springboot -p 8081:8080 --network spring-dev ngotheson/springboot'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
