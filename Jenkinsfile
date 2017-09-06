pipeline {
	agent none

	stages {
		stage('Build') {
			agent { label 'centos'}
			sh 'ant -f build.xml -v'
		}
	}
}
