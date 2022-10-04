node {
    checkout scm
    docker.image('node:lts-buster-slim').inside {
    	stage('Build') {
            sh 'npm install'
    	}
    	stage('Test') {
            sh 'npm run test'
    	}
    }
    stage('Deploy'){
        sh "git push heroku HEAD:master"
    }
}