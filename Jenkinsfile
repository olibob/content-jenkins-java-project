pipeline {
	agent none

	stages {
		stage('Build') {
			agent { label 'centos'}
			steps {
				sh 'ant -f build.xml -v'
			}
		}
	}
}
