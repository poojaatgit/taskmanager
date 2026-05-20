pipeline {
    agent any

    environment {
		DOCKER_IMAGE = "poojaatdocker/taskmanager"
    	DOCKER_TAG = "${BUILD_NUMBER}"
    	JAVA_HOME = "C:\\Program Files\\Java\\jdk-17"
    	MAVEN_HOME = "C:\\Program Files\\Maven\\apache-maven-3.9.15\\apache-maven\\src"
	}

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                echo 'Building JAR with Maven...'
                bat '''
                    set JAVA_HOME=C:\\Program Files\\Java\\jdk-17
                    set PATH=C:\\Program Files\\Java\\jdk-17\\bin;C:\\Program Files\\Maven\\apache-maven-3.9.15\\apache-maven\\src\\bin;%PATH%
                    java -version
                    mvn --version
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat '''
                    set JAVA_HOME=C:\\Program Files\\Java\\jdk-17
                    set PATH=C:\\Program Files\\Java\\jdk-17\\bin;C:\\Program Files\\Maven\\apache-maven-3.9.15\\apache-maven\\src\\bin;%PATH%
                    mvn test
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing to Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                bat "docker stop taskmanager-app || true"
                bat "docker rm taskmanager-app || true"
                bat "docker run -d --name taskmanager-app -p 8080:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}