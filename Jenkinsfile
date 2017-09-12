pipeline {
  agent none

	environment {
		MAJOR_VERSION = '1.0.0'
	}

  stages {
  	stage('say hello') {
  		agent any
  		steps {
  			sayHello 'Bob'
  		}
  	}
  	stage('show sha1') {
  		agent any
  		steps {
  			echo "Branch name: ${env.BRANCH_NAME}"
  			script {
  				def myLib = new myorg.git.gitStuff();

  				echo "My commit: ${myLib.gitCommit(${env.WORKSPACE}/.git)}"
  			}
  		}
  	}
    stage('Unit Tests') {
      agent {
        label 'centos'
      }
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }
    stage('build') {
      agent {
        label 'centos'
      }
      steps {
        sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }
    stage('deploy') {
      agent {
        label 'centos'
      }
      steps {
        sh "mkdir -p /var/www/html/rectangles/${env.BRANCH_NAME}"
        sh "cp dist/rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar /var/www/html/rectangles/${env.BRANCH_NAME}/"
      }
    }
    stage("Running on Master") {
      agent {
        label 'master'
      }
      steps {
        sh "wget http://olibob2.mylabserver.com/rectangles/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage("Test on Debian") {
      agent {
        docker 'openjdk:8u121-jre'
      }
      steps {
        sh "wget http://olibob2.mylabserver.com/rectangles/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage('Promote to green') {
    	agent {
    		label 'centos'
    	}
    	when {
    		branch 'testmaster'

    	}
    	steps {
    		sh "cp /var/www/html/rectangles/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
    	}
    }
    stage('Promote code to testmaster') {
    	agent {
    		label 'master'
    	}
    	when {
    		branch 'bob/FunctionalTesting'
    	}
    	steps {
    		echo 'Stashing any local changes'
    		sh 'git stash'
    		echo 'Checking out the dev branch'
    		sh 'git checkout bob/FunctionalTesting'
    		echo 'Update branch bob/FunctionalTesting'
    		sh 'git pull'
    		echo 'Checking out testmaster'
    		sh 'git checkout testmaster'
    		echo 'Update branch testmaster'
    		sh 'git pull'
    		echo 'Setting username and email locally'
    		sh 'git config  user.email "robby57@gmail.com"'
    		sh 'git config  user.name "Olivier Robert"'
    		echo 'Merging dev into testmaster'
    		sh 'git merge --no-ff bob/FunctionalTesting'
    		echo 'Pushing to Origin testmaster'
    		sh 'git push origin testmaster'
    		echo 'Tagging'
				sh "git tag ${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}"
				echo 'Pushing tag'
				sh "git push origin ${env.MAJOR_VERSION}-build${env.BUILD_NUMBER}"
    	}
				post {
					success {
						emailext(
							subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Dev promoted to Master!",
							body: "<p>${env.JOB_NAME} [${env.BUILD_NUMBER}] Dev promoted to Master!</p><p>Check console output at &QUOT<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOTE</p>",
							to: "orobert@agilepartner.net"
						)
					}
				}
      }
    }
  post {
    failure {
    	emailext(
    		subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] failed!",
    		body: "<p>${env.JOB_NAME} [${env.BUILD_NUMBER}] failed!</p><p>Check console output at &QUOT<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOTE</p>",
    		to: "orobert@agilepartner.net"
    	)
    }
  }
}
