pipeline {
    agent any
    stages {
        stage('Start Timer') {
            steps {
                script {
                    startTime = System.currentTimeMillis()
                }
            }
        }
        stage('Deploy Scenario A') {
            steps {
                sh 'kubectl apply -f scenarios/scenario-a.yaml'
            }
        }
        stage('Wait for Pod to be Running') {
            steps {
                script {
                    timeout(time: 60, unit: 'SECONDS') {
                        waitUntil {
                            def status = sh(script: "kubectl get pod scenario-a -n monitoring -o jsonpath='{.status.phase}'", returnStdout: true).trim()
                            return status == "Running"
                        }
                    }
                }
            }
        }
        stage('Check for Restarts') {
            steps {
                sh 'kubectl describe pod scenario-a -n monitoring > pod-status.txt'
                archiveArtifacts artifacts: 'pod-status.txt'
            }
        }
        stage('End Timer & Log') {
            steps {
                script {
                    endTime = System.currentTimeMillis()
                    duration = (endTime - startTime) / 1000
                    echo "Pipeline Execution Time: ${duration} seconds"
                }
            }
        }
    }
}
