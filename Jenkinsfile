
pipeline {
    agent any

    environment {
        S3_BUCKET = 'microservice.project.artifact.bucket '
    }

    stages {

        stage('Upload Artifact to S3') {
            steps {
                sh '''
                    set -e

                    mkdir -p artifacts

                    tar -czf artifacts/deployment-${BUILD_NUMBER}.tar.gz \
                        Jenkinsfile deployment-service.yml

                    aws s3 cp \
                        artifacts/deployment-${BUILD_NUMBER}.tar.gz \
                        s3://${S3_BUCKET}/microservices-project/${BRANCH_NAME}/${BUILD_NUMBER}/deployment-${BUILD_NUMBER}.tar.gz \
                        --sse AES256
                '''

                archiveArtifacts(
                    artifacts: 'artifacts/*.tar.gz',
                    fingerprint: true
                )
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://959E6939FD0CC72E214840E35699D99A.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh 'kubectl apply -f deployment-service.yml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    serverUrl: 'https://959E6939FD0CC72E214840E35699D99A.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}
