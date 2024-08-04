pipeline {
    agent {
        docker {
            image 'dminus251/jenkins-docker-agent:using_socket' #docker in docker 사용을 위해 docker가 있는 이미지 사용
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
            label 'docker-node-agent'
        }
    }
    environment {
        DOCKER_CREDENTIALS_ID = 'dminus251'
    }
    stages {
        stage ('Check Docker Installation') {
            steps {
                script {
                    sh 'docker --version'
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    sh 'docker build -t dminus251/yyk-server:latest .'
                }
            }
        }
        stage ('Docker Login To Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    }
                }
            }
        }
        stage ('Docker Push') {
            steps {
                script {
                    sh 'docker push dminus251/yyk-server:latest'
                }
            }
        }
	stage ('Docker Run') {
		steps {
			sh 'docker run -d --rm -p 5000:5000 --name yyk-server dminus251/yyk-server:latest'
		}
	}
	stage ('Health Check') {
		steps {
			sh './healthCheck'
			if ($response != '{"status":"ok"}'){
				error("Health Check Failed")
			}
		}
	}
		
        // 나중에 terraform 관련 stage도 추가
    } 
    post {
        always {
                sh "docker stop yyk-server"
        }
    }
}

