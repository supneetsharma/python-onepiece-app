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


        

    }

}
