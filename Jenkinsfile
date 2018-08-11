pipeline {
  agent none

  environment {
  MAJOR_VERSION = 1
  }
    options {
     buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
      }


  stages {
  stage('Say Hello')	{
    agent any
      steps {
        sayHello 'Porcodio'
      }
  }
	stage('Unit Tests') {
		agent {
    	    label'apache'
    	}

        steps {
        sh 'ant -f test.xml -v'
          junit 'reports/result.xml'
        }
      }

	stage('Build') {
		agent {
    	    label 'apache'
    	}

        steps {
        sh 'ant -f build.xml -v'
        }
        post {
         	success {
         	    always {
      			 archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
    		 }
     	}
   }
 }
	stage('Deploy') {
		agent {
    	    label 'apache'
    	}

        steps {
            sh "if ![ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
            sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
			}
		}

	stage('Running on CentOS') {
        agent {
            label 'CentOS'
        }
      	steps {

        sh "wget http://tpavan-d69ca7ed1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"

       }
     }
     stage('Running on Docker Debian'){
         agent{
             docker 'openjdk:10.0-jre'
         }
		steps {

		sh "wget http://tpavan-d69ca7ed1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"

		}
     }
     stage('Promote to Green') {
     	agent {
         label 'apache'
     }
     	when {
     	    branch 'master'
     	}
         steps {
         sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        }
    }
    stage('Promote Dev branch to Master'){
  		agent {
		 label 'apache'
  		}
  		when {
  		    branch 'dev'
  		}
  		steps {
  			echo "identify the user"
  			sh 'whoami'
  			echo "$NODE_NAME is the current node"
  		    echo 'Stashing any local changes'
  		    sh 'git stash'
  		    echo "Checking out dev branch"
  		    sh 'git checkout dev'
  		    echo "git pull origin"
  		    sh 'git pull origin'
  		    echo "Cheking out master branch"
  		    sh 'git checkout master'
  		    //echo "git pull"
  		    //sh 'git pull'
  		    echo "Merging dev into master branch"
  		    sh 'git merge dev'
  		    echo "Pushing to origin master"
  		    sh 'git push origin master'
  		    echo "Tagging the Release"
  		    sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
  		    sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
  		}
  		post {
        	success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "tpavan71@gmail.com"
          )
        }
      }
    }
  }
	  	post {
	    	failure {
	      emailext(
	        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
	        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
	        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
	        to: "tpavan71@gmail.com"
      )
    }
  }
}
