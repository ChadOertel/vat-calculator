pipeline {
    agent any

    parameters {
        string(
            name: 'GIT_REPO_URL',
            defaultValue: 'https://github.com/ChadOertel/vat-calculator.git',
            description: 'Git repository URL to checkout'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                git url: params.GIT_REPO_URL,
                    branch: 'main'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Archive') {
            steps {
                sh 'tar -czf build.tar.gz build'
                archiveArtifacts 'build.tar.gz'
            }
        }
    }
}
