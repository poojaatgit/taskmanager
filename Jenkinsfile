pipeline{
	agent any
	tools{
		jdk 'JDK17'
		maven 'Maven'
	}
	environment{
		DOCKER_IMAGE = "poojaatdocker/taskmanager"
        DOCKER_TAG = "${BUILD_NUMBER}"
	}
	stages{
		stage('Checkout'){
			steps {
                echo 'Pulling code from GitHub...'
                checkout scm
            }
		}
        stage('Check Versions') {
            steps {
                bat '''
                echo JAVA_HOME=%JAVA_HOME%
                where java
                java -version

                where mvn
                mvn -version
                '''
            }
        }
		stage('Build Jar'){
			steps {
                echo 'Building JAR with Maven...'
                bat 'mvn clean package -DskipTests'
            }
		}
		stage('Test') {
            steps {
                echo 'Running unit tests...'
                bat 'mvn test'
            }
        }
		stage('Build Docker Image'){
			steps {
                echo 'Building Docker image...'
                bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
		}
		stage('Push to Docker Hub'){
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
		stage('Run Container') {
            steps {
                echo 'Deploying container locally...'
                bat "docker stop taskmanager-app || true"
                bat "docker rm taskmanager-app || true"
                bat "docker run -d --name taskmanager-app -p 8080:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
	}
	post{
		always {
            cleanWs()
        }
		success{
			echo 'Pipeline completed successfully!'
		}
		failure{
			echo 'Pipeline failed!'
		}
	}
}