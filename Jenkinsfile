pipeline {
    agent any

    parameters {
        string(name: 'SCENARIO', defaultValue: 'a', description: 'Scenario letter (a, b, c, ...)')
    }

    environment {
        START_TIME = ''
        END_TIME = ''
        POD_NAME = "scenario-${params.SCENARIO}"
    }

    stages {
        stage('Start Timer') {
            steps {
                script {
                    START_TIME = System.currentTimeMillis().toString()
                }
            }
        }

        stage('Deploy Scenario') {
            steps {
                sh "kubectl apply -f scenario/scenario-${params.SCENARIO}.yaml"
            }
        }

        stage('Wait for Pod') {
            steps {
                sh '''
                for i in {1..30}; do
                  kubectl get pod $POD_NAME -n monitoring -o jsonpath="{.status.phase}" | grep -q Running && break
                  sleep 2
                done
                '''
            }
        }

        stage('Measure Pod Startup Time') {
            steps {
                script {
                    def podJson = sh(script: "kubectl get pod ${POD_NAME} -n monitoring -o json", returnStdout: true).trim()
                    def pod = readJSON text: podJson

                    def scheduledTime = pod.status.startTime
                    def containerStartTime = pod.status.containerStatuses[0].state.running.startedAt

                    def startupMillis = java.time.Instant.parse(containerStartTime).toEpochMilli() - java.time.Instant.parse(scheduledTime).toEpochMilli()
                    def startupSeconds = startupMillis / 1000

                    echo "🚀 Pod Startup Time for ${POD_NAME}: ${startupSeconds} seconds"
                }
            }
        }

        stage('Check Restarts') {
            steps {
                script {
                    def restarts = sh(
                        script: "kubectl get pod ${POD_NAME} -n monitoring -o jsonpath='{.status.containerStatuses[0].restartCount}'",
                        returnStdout: true
                    ).trim()
                    echo "🔁 Pod Restart Count for ${POD_NAME}: ${restarts}"
                }
            }
        }

        stage('End Timer & Log') {
            steps {
                script {
                    END_TIME = System.currentTimeMillis().toString()
                    def durationSeconds = (END_TIME.toLong() - START_TIME.toLong()) / 1000
                    echo "⏱ Scenario ${params.SCENARIO.toUpperCase()} Total Pipeline Time: ${durationSeconds} seconds"
                }
            }
        }
    }
}
