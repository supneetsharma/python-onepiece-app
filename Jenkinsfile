pipeline {
    
    agent any

    stages {

        stage("Git checkout") {
            steps {
                echo "Cloniing the Pythonn project source code from the GitHub repository"
                git branch: 'main', url: 'https://github.com/falakdesai/python-onepiece-app'
            }
        }
        
        stage("python build") {
            steps {
                echo "Building the Python application"
                sh '''
                rm -rf venv # Remove existing virtual environment if any
                python -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage("Python test") {
            steps {
                echo "Running tests for the Python application"
                sh '''
                . venv/bin/activate
                pytest app/tests/test_app.py
                '''
            }
        }


        stage("sonarQube scan") {
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' }
            steps {
                echo "Running SonarQube scan for code quality analysis"
                withSonarQubeEnv('sonarqube-local') {
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    '''
                }
            }
        }

        stage("publish sonarQube quality gate") {
            environment { SONARQUBE_SCANNER_HOME = tool 'sonar-scanner' }
            steps {
                echo "Re-running SonarQube scan for code quality analysis"
                withSonarQubeEnv('sonarqube-local') {
                    sh '''
                    . venv/bin/activate
                    $SONARQUBE_SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=Jenkins \
                    -Dsonar.sources=. \
                    -Dsonarqualitygate.wait=false
                    '''
                }
            }
        }

        stage("Docker build and push") {
            steps {
                echo "Building and Pushing the Docker image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh '''
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    docker build -t falakdesai2/python-onepiece-app:latest .
                    docker push falakdesai2/python-onepiece-app:latest
                    '''
                }
            }
        }

        stage("Trivy security scan") {
            steps {
                echo "Performing security scan using Trivy"
                sh '''
                trivy image --format table --severity HIGH,CRITICAL \
                --output trivy-report.txt falakdesai2/python-onepiece-app:latest
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }

        

    }

}
