#!/usr/bin/groovy

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

	environment {
		PYTHONPATH = "${WORKSPACE}"
		PATH = "C:\\Program Files\\Git\\usr\\bin;C:\\Program Files\\Git\\bin;${env.PATH}"
	}

    stages {

		stage("Test - Unit tests") {
			steps { runUnittests() }
		}

        stage("Build") {
            steps { buildApp() }
		}

        stage("Deploy - Dev") {
            steps { deploy('dev') }
		}

		stage("Test - UAT Dev") {
            steps { runUAT(8880) }
		}

        stage("Deploy - Stage") {
            steps { deploy('stage') }
		}

		stage("Test - UAT Stage") {
            steps { runUAT(8800) }
		}

        stage("Approve") {
            steps { approve() }
		}

        stage("Deploy - Live") {
            steps { deploy('live') }
		}

		stage("Test - UAT Live") {
            steps { runUAT(80) }
		}

	}
}


// steps
def buildApp() {
		def appImage = docker.build("hands-on-jenkins/myapp:${BUILD_NUMBER}")
		docker.withRegistry('0.0.0.0:5000', 'Global'){
			appImage.push();
		}
}


def deploy(environment) {

	def containerName = ''
	def port = ''

	if ("${environment}" == 'dev') {
		containerName = "app_dev"
		port = "8880"
	} 
	else if ("${environment}" == 'stage') {
		containerName = "app_stage"
		port = "8800"
	}
	else if ("${environment}" == 'live') {
		containerName = "app_live"
		port = "80"
	}
	else {
		println "Environment not valid"
		System.exit(0)
	}

	sh "docker ps -f name=${containerName} -q | xargs --no-run-if-empty docker stop"
	sh "docker ps -a -f name=${containerName} -q | xargs -r docker rm"
	sh "docker run -d -p ${port}:5000 --name ${containerName} hands-on-jenkins/myapp:${BUILD_NUMBER}"

}


def approve() {

	timeout(time:1, unit:'DAYS') {
		input('Do you want to deploy to live?')
	}

}


def runUnittests() {
	sh "pip3 install --no-cache-dir -r ./requirements.txt"
	sh "python tests/test_flask_app.py"
}


def runUAT(port) {
	sh "tests/runUAT.sh ${port}"
}
