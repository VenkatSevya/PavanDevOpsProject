pipeline {
  agent any

  tools {
    // Install the Maven version configured as "M2_HOME" and add it to the path.
    maven "M2_HOME"
  }
  environment{
	AWS_ACCOUNT_ID = "962826464166" 
	AWS_DEFAULT_REGION = "us-east-1"
	IMAGE_REPO_NAME = "webapp"
	IMAGE_TAG = "latest"
	REPOSIT_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazon.aws.com/${IMAGE_REPO_NAME}"
  }

  stages {
    
  stage('GIT Clone') {
            steps {
                checkout([
                 $class: 'GitSCM',
                 branches: [[name: 'main']],
                 userRemoteConfigs: [[
                    url:  "https://github.com/PavanDeepakPagadala/PavanDevOpsProject.git",
                    credentialsId: '',
                 ]]
                ])
            }
        }
	stage('Build') {
	     steps {
	         sh "mvn clean  package"
	     }
	}
	stage('Test') {
	    steps {
	        sh "mvn clean verify -DskipITs=true',junit '**/target/surefire-reports/TEST-*.xml'archive 'target/*.jar"

	    }
	}
	//stage('SonarQube analysis') {
    //def scannerHome = tool 'SonarScanner 4.7';
      //  steps{
       // withSonarQubeEnv('sonarqube') { 
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        //sh "mvn sonar:sonar"
   // }
      //  }
       // }
	//TO upload war file into S3 bucket using IAM role and S3Profile configuration in jenkins.
	 stage('Upload To S3 Bucket ') {
      steps {
        s3Upload consoleLogLevel: 'INFO',
	dontSetBuildResultOnFailure: false,
	dontWaitForConcurrentBuildCompletion: false,
	entries: [
				[
					bucket: 'testbucketpav', 
					excludedFile: '/webapp/target',
					flatten: false,
					gzipFiles: false,
					keepForever: false,
					managedArtifacts: false,
					noUploadOnFailure: false,
					selectedRegion: 'us-east-1',
					showDirectlyInBrowser: false,
					sourceFile: '**/webapp/target/*.war',
					storageClass: 'STANDARD',
					uploadFromSlave: false,
					useServerSideEncryption: false
				]
			], 
	pluginFailureResultConstraint: 'FAILURE', 
	profileName: 'S3Profile', //Give same name as configured in jenkins
	userMetadata: []

      }
    }
	//To remove old war files
	stage('Clean'){
		steps{
			sh "sudo rm -f /opt/tomcat/webapps/webapp.war" 
		}
	}
	//To download war files from s3 bucket to tomcat 
	stage('Deploy to Tomcat from S3') {
	    steps {   
			

	        sh " sudo aws s3 cp s3://testbucketpav/webapp/target/webapp.war /opt/tomcat/webapps/" 
	    }
	}
	//For Email notification on build status
	stage('Email'){
		steps {
		emailext body: '$DEFAULT_CONTENT', //configure message in body in jenkins
		 subject: 'Jenkins Build Status', 
		 to: 'pavandeepakpagadala@gmail.com'
		}
	}
	//TO build docker image and push to AWS ECR repository by taking war file from S3 bucket
	//Use docker context use command inside job directory to build files using Dockerfile
	stage('Publish to ECR') {
		steps {
			script {
				sh "sudo aws s3 cp s3://testbucketpav/webapp/target/webapp.war /var/lib/jenkins/workspace/Java_Pipeline_Application/" //download war file from s3 to job do=irectory for build
				sh "pwd" //to know current directory
				sh "sudo aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 962826464166.dkr.ecr.us-east-1.amazonaws.com"
				 sh "sudo docker build -t webapp ." //to build war file from present directory with tag webapp
				sh "sudo docker tag webapp:latest 962826464166.dkr.ecr.us-east-1.amazonaws.com/webapp:latest"
				sh "sudo docker push 962826464166.dkr.ecr.us-east-1.amazonaws.com/webapp:latest"
			}
		}
	}
	
  

  }

}
