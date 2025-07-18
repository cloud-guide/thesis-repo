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
                    env.START_TIME_MS = System.currentTimeMillis().toString()
                }
            }
        }

        stage('Deploy Scenario') {
            steps {
                script {
                    // Delete old pod if exists
                    sh """
                    kubectl delete pod ${POD_NAME} -n monitoring --ignore-not-found
                    kubectl apply -f scenario/scenario-${params.SCENARIO}.yaml
                    """
                }
            }
        }

        stage('Wait for Pod') {
            steps {
                script {
                    def maxTries = 30
                    def waitSuccess = false

                    for (int i = 0; i < maxTries; i++) {
                        def phase = sh(script: "kubectl get pod $POD_NAME -n monitoring -o jsonpath='{.status.phase}'", returnStdout: true).trim()
                        echo "🔄 Attempt ${i + 1}: Pod phase is '${phase}'"
                        if (phase == "Running") {
                            waitSuccess = true
                            break
                        }
                        sleep 2
                    }

                    if (!waitSuccess) {
                        echo "⚠️ Pod $POD_NAME did not reach Running state. Continuing anyway to fetch restart count."
                    }
                }
            }
        }

        stage('Measure Pod Startup Time') {
            steps {
                script {
                    try {
                        def podJson = sh(script: "kubectl get pod ${POD_NAME} -n monitoring -o json", returnStdout: true).trim()
                        def pod = readJSON text: podJson

                        def scheduledTime = pod.status.startTime
                        def containerStatuses = pod.status.containerStatuses

                        def containerStartTime = null
                        for (cs in containerStatuses) {
                            if (cs.state?.running?.startedAt) {
                                containerStartTime = cs.state.running.startedAt
                                break
                            }
                        }

                        if (scheduledTime && containerStartTime) {
                            def startupMillis = java.time.Instant.parse(containerStartTime).toEpochMilli() - java.time.Instant.parse(scheduledTime).toEpochMilli()
                            def startupSeconds = startupMillis / 1000
                            echo "🚀 Pod Startup Time for ${POD_NAME}: ${startupSeconds} seconds"
                        } else {
                            echo "⚠️ Could not determine exact startup time (some containers may not have started)"
                        }
                    } catch (err) {
                        echo "⚠️ Error measuring startup time: ${err.getMessage()}"
                    }
                }
            }
        }

        stage('Check Restarts') {
            steps {
                script {
                    try {
                        def podName = POD_NAME
                        def restartsJson = sh(
                            script: "kubectl get pod ${podName} -n monitoring -o json",
                            returnStdout: true
                        ).trim()

                        def pod = readJSON text: restartsJson
                        def totalRestarts = 0
                        for (cs in pod.status.containerStatuses) {
                            echo "🔁 Container '${cs.name}' Restart Count: ${cs.restartCount}"
                            totalRestarts += cs.restartCount
                        }

                        echo "🔁 Total Restart Count for ${podName}: ${totalRestarts}"
                    } catch (err) {
                        echo "⚠️ Error checking restarts: ${err.getMessage()}"
                    }
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
