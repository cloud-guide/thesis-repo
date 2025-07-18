pipeline {
    agent any

    parameters {
        string(name: 'SCENARIO', defaultValue: 'a', description: 'Scenario letter (a, b, c, ...)')
    }

    environment {
        POD_NAME = "scenario-${params.SCENARIO}"
    }

    stages {
        stage('Start Timer') {
            steps {
                script {
                    // Use script-scoped variables
                    env.START_TIME_MS = System.currentTimeMillis().toString()
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
                script {
                    def maxTries = 30
                    def waitSuccess = false

                    for (int i = 0; i < maxTries; i++) {
                        def status = sh(script: "kubectl get pod $POD_NAME -n monitoring -o jsonpath='{.status.phase}'", returnStdout: true).trim()
                        if (status == "Running") {
                            waitSuccess = true
                            break
                        }
                        sleep 2
                    }

                    if (!waitSuccess) {
                        error("Pod $POD_NAME did not reach Running state in time.")
                    }
                }
            }
        }

        stage('Measure Pod Startup Time') {
            steps {
                script {
                    def podJson = sh(script: "kubectl get pod ${POD_NAME} -n monitoring -o json", returnStdout: true).trim()
                    def pod = readJSON text: podJson

                    def scheduledTime = pod.status.startTime

                    def containerState = pod.status.containerStatuses[0].state
                    if (!containerState.running) {
                        error("Container is not in running state yet.")
                    }

                    def containerStartTime = containerState.running.startedAt

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
                    def endTime = System.currentTimeMillis()
                    def startTime = env.START_TIME_MS.toLong()
                    def durationSeconds = (endTime - startTime) / 1000
                    echo "⏱ Scenario ${params.SCENARIO.toUpperCase()} Total Pipeline Time: ${durationSeconds} seconds"
                }
            }
        }
    }
}
