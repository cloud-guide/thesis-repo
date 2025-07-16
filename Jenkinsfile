pipeline {
    agent any

    environment {
        START_TIME = ''
        END_TIME = ''
    }

    stages {
        stage('Start Timer') {
            steps {
                script {
                    START_TIME = System.currentTimeMillis()
                }
            }
        }

        stage('Deploy Scenario A') {
            steps {
                sh 'kubectl apply -f scenario/scenario-a.yaml'
            }
        }

        stage('Wait for Pod') {
            steps {
                sh '''
                for i in {1..30}; do
                  kubectl get pod scenario-a -o jsonpath="{.status.phase}" | grep -q Running && break
                  sleep 2
                done
                '''
            }
        }

        stage('Check Restarts') {
            steps {
                sh 'kubectl get pod scenario-a -o jsonpath="{.status.containerStatuses[0].restartCount}"'
            }
        }

        stage('End Timer & Log') {
            steps {
                script {
                    END_TIME = System.currentTimeMillis()
                    def durationSeconds = (END_TIME.toLong() - START_TIME.toLong()) / 1000
                    echo "⏱ Scenario A Total Time: ${durationSeconds} seconds"
                }
            }
        }
    }
}
