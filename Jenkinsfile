pipeline {
    agent any

    stages {
       stage('Clone Repo') {
    steps {
        git branch: 'main', url: 'https://github.com/cloud-guide/thesis-repo.git'
    }
}

        stage('Build') {
            steps {
                echo 'Building the application...'
                sleep 10
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sleep 5
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to test environment...'
                sleep 5
            }
        }
    }
}
