pipeline {
	agent { label 'centos' }

	options {
		disableConcurrentBuilds()
		skipStagesAfterUnstable()
		timestamps()
		buildDiscarder(logRotator(numToKeepStr: '5', artifactDaysToKeepStr: '2'))
	}

	stages {
		stage('Build') {

			steps {
				sh 'ant -f build.xml -v'
			}
		}
	}

	post {
		always {
			archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
		}
	}
}
