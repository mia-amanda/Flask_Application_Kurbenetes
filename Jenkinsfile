pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "${params.DOCKER_USERNAME}/flask-app:latest"
        JIRA_SITE = 'Jira-Instance' // Replace with your Jira instance URL
        JIRA_PROJECT_KEY = 'YourProjectKey' // Replace with your Jira project key
        DOCKER_USERNAME = credentials('docker-hub-credentials')?.USERNAME
        DOCKER_PASSWORD = credentials('docker-hub-credentials')?.PASSWORD
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build and push Docker image
                    bat """
                    docker build -t ${DOCKER_IMAGE} .
                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                """
            }
        }

        stage('Monitor Pods') {
            steps {
                script {
                    def podStatus = bat(returnStdout: true, script: 'kubectl get pods').trim()

                    // Check for any pods in Error/CrashLoopBackOff status
                    if (podStatus.contains('Error') || podStatus.contains('CrashLoopBackOff')) {
                        currentBuild.result = 'UNSTABLE'

                        // Create a Jira ticket if any pod is unhealthy
                        jiraNewIssue site: env.JIRA_SITE,
                                     projectKey: env.JIRA_PROJECT_KEY,
                                     summary: "Pod Failure Detected in Kubernetes Cluster",
                                     description: """
                                     There was an issue detected with the following pod(s):
                                     ${podStatus}
                                     Please investigate the issue immediately.
                                     """,
                                     issueType: 'Bug'
                    } else {
                        echo "All Pods are running fine."
                    }
                }
            }
        }

        stage('Check Minikube Service') {
            steps {
                bat """
                minikube service flask-service --url
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
