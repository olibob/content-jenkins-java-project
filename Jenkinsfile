pipeline {
  agent none

  stages {
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
        sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/${env.BRANCH_NAME}/"
      }
    }
    stage("Running on Master") {
      agent {
        label 'master'
      }
      steps {
        sh "wget http://olibob2.mylabserver.com/rectangles/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage("Test on Debian") {
      agent {
        docker 'openjdk:8u121-jre'
      }
      steps {
        sh "wget http://olibob2.mylabserver.com/rectangles/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
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
    		sh "cp /var/www/html/rectangles/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
    	}
    }
    stage('Promote code to testmaster') {
    	agent {
    		label 'centos'
    	}
    	when {
    		branch 'bob/FunctionalTesting'
    	}
    	steps {
    		echo 'Stashing any local changes'
    		sh 'git stash'
    		echo 'Checking out the dev branch'
    		sh 'git checkout bob/FunctionalTesting'
    		echo 'Checking out testmaster'
    		sh 'git checkout testmaster'
    		echo 'Setting username and email locally'
    		sh 'git config --local user.email "robby57@gmail.com"
    		sh 'git config --local user.name "Olivier Robert"
    		echo 'Merging dev into testmaster'
    		sh 'git merge --no-ff bob/FunctionalTesting'
    		echo 'Pushing to Origin testmaster'
    		sh 'git push origin testmaster'
    	}
    }
  }
}
