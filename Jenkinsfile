pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Building"'
		sh 'echo "Testing WeebHook"'
            }
	}
        stage('Test') {
            steps {
                sh 'echo "Testing"'
            }
	}
	stage('Deploy') {
            steps {
                sh 'echo "Deploying"'
            }
        }
    }
}
