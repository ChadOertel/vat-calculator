pipeline { 
    agent any 
    environment {
        dockerCreds = credentials('dockerhub_login')
        registry = "${dockerCreds_USR}/vatcal"
        registryCredentials = "dockerhub_login"
        dockerImage = ""
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
