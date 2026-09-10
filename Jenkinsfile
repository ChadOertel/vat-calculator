pipeline { 
    agent any 
    environment {
        gcpCreds = 'gcp_credentials'
        dockerCreds = credentials('dockerhub_login')
        registry = "${dockerCreds_USR}/vatcal"
        registryCredentials = "dockerhub_login"
        dockerImage = ""
        TF_VAR_gcp_project = "qwiklabs-gcp-02-e5425ade991f"
        TF_VAR_docker_registry = "${registry}"
    }
 
    stages { 
        stage('Run Tests') { 
            steps { 
                sh 'npm install' 
                sh 'CI=true npm test' 
             } 
        } 


        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build(registry)
                    dockerImage.tag("${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Grype Scan') {
            steps {
                grypeScan scanDest: "docker:${registry}:${env.BUILD_NUMBER}", repName: "scanResult.txt", autoInstall: true
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("", registryCredentials) {
                        dockerImage.push('latest')
                        dockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Clean Up') {
            steps {
                sh "docker image prune --all --force --filter 'until=48h'"
            }
        }

        stage('Provision Server') {
            steps {
                script {
                    withCredentials([file(credentialsId: gcpCreds, variable:'GCP_CREDENTIALS')]) {
                        sh '''
                        export GOOGLE_APPLICATION_CREDENTIALS=$GCP_CREDENTIALS
                        terraform init
                        terraform apply -auto-approve
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'scanResult.txt', allowEmptyArchive: true
            // recordIssues(
            //     qualityGates: [
            //         [criticality: 'FAILURE', integerThreshold: 30, threshold: 30.0, type: 'TOTAL_HIGH'], 
            //         [criticality: 'FAILURE', integerThreshold: 5, threshold: 5.0, type: 'NEW']
            //         ], 
            //         sourceCodeRetention: 'LAST_BUILD', 
            //         tools: [grype()]
            // )
        }
    }
}
