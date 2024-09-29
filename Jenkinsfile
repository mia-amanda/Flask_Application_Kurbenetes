pipeline {
    agent any

    environment {
        JIRA_SITE = 'sit-team-vr03pn0q' // Your Jira site name
        JIRA_PROJECT_KEY = 'PMP' // Your Jira project key
    }

    stages {
        stage('Checkout') {
            steps {
                timeout(time: 5, unit: 'MINUTES') { // Extend the timeout to 5 minutes
                    deleteDir()
                    git branch: 'main', url: 'https://github.com/mia-amanda/Flask_Application_Kurbenetes.git'
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build and push Docker image
                    def dockerCredentials = credentials('docker-hub-credentials')
                    def dockerUsername = dockerCredentials.username
                    def dockerPassword = dockerCredentials.password

                    bat """
                    docker build -t ${dockerUsername}/flask-app:latest .
                    docker login -u ${dockerUsername} -p ${dockerPassword}
                    docker push ${dockerUsername}/flask-app:latest
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
