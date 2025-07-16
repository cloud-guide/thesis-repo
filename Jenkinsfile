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
                    START_TIME = System.currentTimeMillis().toString()
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
                  kubectl get pod scenario-a -n monitoring -o jsonpath="{.status.phase}" | grep -q Running && break
                  sleep 2
                done
                '''
            }
        }

        stage('Measure Pod Startup Time') {
            steps {
                script {
                    def podJson = sh(script: 'kubectl get pod scenario-a -n monitoring -o json', returnStdout: true).trim()
                    def pod = readJSON text: podJson

                    def scheduledTime = pod.status.startTime
                    def containerStartTime = pod.status.containerStatuses[0].state.running.startedAt

                    def startupMillis = java.time.Instant.parse(containerStartTime).toEpochMilli() - java.time.Instant.parse(scheduledTime).toEpochMilli()
                    def startupSeconds = startupMillis / 1000

                    echo "🚀 Pod Startup Time: ${startupSeconds} seconds"
                }
            }
        }

        stage('Check Restarts') {
            steps {
                script {
                    def restarts = sh(
                        script: 'kubectl get pod scenario-a -n monitoring -o jsonpath="{.status.containerStatuses[0].restartCount}"',
                        returnStdout: true
                    ).trim()
                    echo "🔁 Pod Restart Count: ${restarts}"
                }
            }
        }

        stage('End Timer & Log') {
            steps {
                script {
                    END_TIME = System.currentTimeMillis().toString()
                    def durationSeconds = (END_TIME.toLong() - START_TIME.toLong()) / 1000
                    echo "⏱ Scenario A Total Pipeline Time: ${durationSeconds} seconds"
                }
            }
        }
    }
}
