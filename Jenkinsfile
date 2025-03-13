// example pipeline for java maven spring-boot app
pipeline {
  agent any 	
  //give a hoot—don't pollute!
  options {
    buildDiscarder(logRotator(daysToKeepStr: '5', artifactNumToKeepStr: '10'))
    disableConcurrentBuilds()
  }

  //environment vars can be used for various purposes
  //if you fork this repo you need to do the following actions: 
  //1. Nothing to do or change to download the code to the Jenkins workspace ( it is automatic )
  //2. Make sure you can access the SONARQUBEHOST
  //   - requires an organizational level token  to do the scan and service account to access the dashboard.
  //   - open an request at: https://github.dxc.com/AET/SonaraaS/issues
  //4. Change the ARTIFACTORYPATH in your project's maven pom.xml file
  //      - Artifactory Repo, URL should be https://artifactory.dxc.com/artifactory/<YOUR_MAVEN_REPO_NAME>
  //      - Make sure the Artifactory Repo is of type MAVEN
  //5. Add your projects' ARIFACTORY r/w service account id and password to your project's Jenkins credentials vault
  //6. Update the JENKINSCREDID in the environment section to match the crendentials id used to save the Artifactory credentials in jenkins. 
  //7. Update the Maven pom.xml file with your applications information (e.g. groupId and artifactId) 
	  	
  environment {
    //PACKAGE_PATH = "helloworld"
    PACKAGE_VERSION_PREFIX = "0.0.1"
    SONARHOST = "https://usplsvulx1005.elabs.svcs.entsvcs.com/"
    SONARTOKEN = "55b18f5c5277897462cb18cfb8cc7a117e7aece1"
    JENKINSCREDID  = "central"	 
    ARTIFACTORYPATH = "https://artifactory.dxc.com/artifactory/eprocurement-maven"	 
    DXC_ARTIFACTORY_URL="docker.dxc.com/eprocurement-docker"  
    VERSION = "latest"		
    IMAGE="eproc-indent-service" 
    //MAVEN_SETTINGS = '<settings xmlns=\\"http://maven.apache.org/SETTINGS/1.0.0\\" xmlns:xsi=\\"http://www.w3.org/2001/XMLSchema-instance\\" xsi:schemaLocation=\\"http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd\\"><servers><server><username>svc-eprocurement-maven</username><password>Cw23@wg45</password><id>central</id></server><server><username>svc-eprocurement-maven</username><password>Cw23@wg45</password><id>snapshots</id></server></servers><profiles><profile><repositories><repository><id>central1</id><name>Maven Repository Switchboard</name><url>https://repo.maven.apache.org/maven2</url></repository><repository><id>central2</id><name>Maven Repository Switchboard</name><url>https://maven.repository.redhat.com/ga</url></repository><repository><snapshots><enabled>false</enabled></snapshots><id>central</id><name>eprocurement-maven</name><url>https://artifactory.dxc.com:443/artifactory/eprocurement-maven</url></repository><repository><snapshots /><id>snapshots</id><name>eprocurement-maven</name><url>https://artifactory.dxc.com:443/artifactory/eprocurement-maven</url></repository></repositories><pluginRepositories><pluginRepository><id>central1</id><name>Maven Repository Switchboard</name><url>https://repo.maven.apache.org/maven2</url></pluginRepository><pluginRepository><snapshots><enabled>false</enabled></snapshots><id>central</id><name>eprocurement-maven</name><url>https://artifactory.dxc.com:443/artifactory/eprocurement-maven</url></pluginRepository><pluginRepository><snapshots /><id>snapshots</id><name>eprocurement-maven</name><url>https://artifactory.dxc.com:443/artifactory/eprocurement-maven</url></pluginRepository></pluginRepositories><id>artifactory</id></profile></profiles><activeProfiles><activeProfile>artifactory</activeProfile></activeProfiles></settings>'
    //Smart Scan Threshold limit for Malware, Vulnerability, Contents & Checklists
    MALWARE_THRESHOLD =               "\"malware\": 10"
    VULNERABILITY_THRESHOLD = "\"vulnerabilities\": {\"defcon1\":10, \"critical\":10, \"high\":10}"
    CONTENTS_THRESHOLD =             "\"contents\": {\"defcon1\":10, \"critical\":10, \"high\":10}"
    CHECKLISTS_THRESHOLD =         "\"checklists\": {\"defcon1\":10, \"critical\":10, \"high\":10}"  
  }
   
  stages{
    stage('Prepare workspace') {             
      steps { 
        sh "echo Installing JDK 15 and Maven 3 on slave server"
        sh '''
          wget https://download.java.net/java/GA/jdk15.0.2/0d1cfde4252546c6931946de8db48ee2/7/GPL/openjdk-15.0.2_linux-x64_bin.tar.gz 
          rm -rf /usr/java
          mkdir /usr/java
          cp openjdk-15.0.2_linux-x64_bin.tar.gz /usr/java
          tar -xzvf /usr/java/openjdk-15.0.2_linux-x64_bin.tar.gz --directory /usr/java/
          JAVA_HOME=/usr/java/jdk-15.0.2
          export JAVA_HOME
          echo $JAVA_HOME
	  export PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
	  export PATH
	  echo $PATH
	  update-alternatives --install "/usr/bin/java" "java" "/usr/java/jdk-15.0.2/bin/java" 1
	  update-alternatives --install "/usr/bin/javac" "javac" "/usr/java/jdk-15.0.2/bin/javac" 1	
	  update-alternatives --set java $JAVA_HOME/bin/java
	  update-alternatives --set javac $JAVA_HOME/bin/javac
	  java -version
          rm -rf ~/build/software/maven/
          rm -f /usr/sbin/mvn
	  mkdir -p ~/build/software/maven/
	  wget https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz --directory ~/build/software/maven/
	  tar -xvf ~/build/software/maven/apache-maven-3.6.3-bin.tar.gz --directory ~/build/software/
          ln -s ~/build/software/apache-maven-3.6.3/bin/mvn /usr/sbin/mvn           
	'''
      }
    }
    	    	    
    stage('Clean Workspace') {             
      steps { 
        sh "echo Build and Package"
        sh "mvn -B -DskipTests clean package -s settings1.xml"
      }
    }
        
    // static code analysis using sonarqube
    stage('Static Analysis - SonarQube') {
      steps {
        sh 'echo static analysis'
        //sh "mvn jacoco:report sonar:sonar -Dsonar.host.url=$SONARHOST -Dsonar.login=$SONARTOKEN"
        //archiveArtifacts artifacts: '**/target/sonar/report-task.txt'    
      }
    }
    
    //quality gate for sonarqube
    stage('Quality Gate - SonarQube') {
      steps {  
        sh 'echo quality gate'   
	//sh 'bash -x gate.sh $SONARHOST $SONARTOKEN'
      }
    }	    
		    
    //  if the dependacy checker central servers can be a bit flakey.. 
    //  so this can cause issues that are outside of the project control
    //  the current workaround is try to do the build later or comment out the checker..
    //  note you will also need to update the pom.xml file	
    stage('Dependency Check - OWASP') {
      steps {
        sh 'echo dependancy check'
        sh "mvn --batch-mode dependency-check:check -C"
      }
      post {
        always {
    	      publishHTML(target:[
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target',
            reportFiles: 'dependency-check-report.html',
            reportName: "OWASP Dependency Check Report"
          ])
        }
      }
    } // end stage 	   
     
    //saves Jenkins workspace to store test results
    //Archiving artifacts this way is not a substitute for using external artifact repositories such as Artifactory  
    //should be considered only for basic reporting and debugging 
    stage('Save Jenkins Workspace'){
      steps {
        sh 'echo store jenkins workspace'
        //sh "mvn site"    
        //archiveArtifacts artifacts: '**/target/site/**/*'    
        //archiveArtifacts artifacts: '**/target/dependency-check*.*' 
        //archiveArtifacts artifacts: '**/target/jacoco-ut/jacoco.xml' 
        // archiveArtifacts artifacts: '**/target/site/surefire-report.html'
      }
    }        
        
    // the resulting package created from running an automated build should always be stored in a binary repository ( Artifactory )
    // with the intial package status of "unstable".  Only after the unstable package passes integration, api and/or functional testing 
    // should the package status be marked as "stable" in Artifactory.
    // only runs on the 'master' branch     
    stage('Push to Artifactory - JFrog eproc-maven'){
      when {
	branch 'develop'    
  	environment name: 'GIT_URL', value: 'https://github.dxc.com/eProcurement-V2/eproc-indent-service.git'
      }	
      steps {
        sh 'echo store to Artifactory'
	//sh 'printenv'
        //store automated build artifacts into Artifactory
        //the Artifactory credentials are stored  within Jenkins credentials
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.JENKINSCREDID, usernameVariable: 'A_USERNAME', passwordVariable: 'A_PASSWORD']]){
        sh "echo '<settings xmlns=\"http://maven.apache.org/SETTINGS/1.0.0\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xsi:schemaLocation=\"http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd\"><servers><server><id>$JENKINSCREDID</id><username>$A_USERNAME</username><password>$A_PASSWORD</password><filePermissions>664</filePermissions><directoryPermissions>775</directoryPermissions><configuration></configuration></server></servers></settings>' > settings2.xml"
        sh "mvn -DskipTests deploy -s settings2.xml" 
        }         
      }
    }
        
    // send a build pipeline messages to MS Teams    
    stage('Send Message - MS Teams'){
      steps {
        sh "echo send message"
        //sending messages to MS Teams does not require a special stage
        //From MS Teams add the enterprise github connector to your channel and follow the direction to add the webhook to github
        //After this is setup and configured you MS Team channel will receive message from github and jenkins.                  
      } //steps
    } // stage
        
    stage('Verify Artifacts - JFrog Maven'){
      steps {
        sh "echo ArtifactsCheck"
        sh "pwd"
        sh "ls -ltra"
        sh "ls -ltra target/"  
        //sh "ls -ltra target/sonar" 
        sh "cp target/indent-service-0.0.1-SNAPSHOT.jar indent-service.jar"         
      } //steps
    } 

    // Docker build 
    stage('Build Docker Image - Local'){
      steps {
        script {
          withCredentials ([
            usernamePassword(
              credentialsId: 'central',
              usernameVariable: 'ARTIFACTORY_USER', 
              passwordVariable: 'ARTIFACTORY_PASSWORD')
          ]) {
          sh '''
            docker login docker.dxc.com/eprocurement-docker -u ${ARTIFACTORY_USER} -p ${ARTIFACTORY_PASSWORD}
            docker build -f Dockerfile -t $DXC_ARTIFACTORY_URL/$IMAGE:$VERSION .
          '''
	  }
        } //script
      } //steps
    } //stage
    
    /*stage("Docker Image Scan - Trend Micro Smart Scan") {
      steps {
        script {
          withCredentials ([
            usernamePassword(
              credentialsId: 'smartcheck',
              usernameVariable: 'SMARTCHECK_USER', 
              passwordVariable: 'SMARTCHECK_PASSWORD'),
            usernamePassword(
              credentialsId: 'preregistry',
              usernameVariable: 'PREREGISTRY_USER', 
              passwordVariable: 'PREREGISTRY_PASSWORD')
          ]) {
              echo 'scanning the image'
              sh '''      
                 echo '{ \"insecure-registries\" : [\"localhost:32000\", \"a850227dd8c6f4293ac7ff04ca66e20f-740358102.ap-south-1.elb.amazonaws.com:5000\"] }' > /etc/docker/daemon.json
                 systemctl restart docker
                 docker run -v /var/run/docker.sock:/var/run/docker.sock -v /etc/docker/daemon.json:/etc/docker/daemon.json  deepsecurity/smartcheck-scan-action --image-name $DXC_ARTIFACTORY_URL/$IMAGE:$VERSION --smartcheck-host="a850227dd8c6f4293ac7ff04ca66e20f-740358102.ap-south-1.elb.amazonaws.com" --smartcheck-user $SMARTCHECK_USER --smartcheck-password $SMARTCHECK_PASSWORD --insecure-skip-tls-verify --insecure-skip-registry-tls-verify --preregistry-scan --preregistry-user $PREREGISTRY_USER --preregistry-password $PREREGISTRY_PASSWORD  --findings-threshold "{ $MALWARE_THRESHOLD, $VULNERABILITY_THRESHOLD, $CONTENTS_THRESHOLD, $CHECKLISTS_THRESHOLD}"
              '''
              } //sh
        } //script
      } //steps
    }//stage */
    stage('Push to Artifactory - JFrog eproc-docker') {
      when {
        branch 'develop'    
  	environment name: 'GIT_URL', value: 'https://github.dxc.com/eProcurement-V2/eproc-indent-service.git'
      }
        steps {
          script {
            withCredentials ([
              usernamePassword(
                credentialsId: 'central',
                usernameVariable: 'ARTIFACTORY_USER', 
                passwordVariable: 'ARTIFACTORY_PASSWORD')
            ]) {
            sh '''             
	      docker login docker.dxc.com/eprocurement-docker -u ${ARTIFACTORY_USER} -p ${ARTIFACTORY_PASSWORD}
	      docker push $DXC_ARTIFACTORY_URL/$IMAGE:$VERSION
            '''
            } // sh
          } // script
        } //steps
    }//stage
	    
    stage ('Logout from JFrog') {
      steps {
        sh "docker logout docker.dxc.com/eprocurement-docker"
      }
    }     	      
  }//stages

  post {
    always {
      echo 'One way or another, I have finished'	
      deleteDir() /* clean up our workspace */
    }
    success {
      echo 'I succeeded!'
    }
    unstable {
      echo 'I am unstable :/'
    }
    failure {
      echo 'I failed :('
    }
    changed {
      echo 'Things were different before...'
    }
  } //post
} //pipeline
