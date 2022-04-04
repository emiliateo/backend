def gradlew(String... args) {
    sh "./gradlew ${args.join(' ')} -s"
}

pipeline {
  agent any
    
  tools {
		gradle "gradle"
		}
  environment {
        BUILD_DATE = sh(returnStdout: true, script: "date -u +'%d-%m-%Y-%H-%M-%S'").trim()
        DB_URL = "mydockerdatabase.cqnuqytrqlro.us-east-1.rds.amazonaws.com"
		DB_PORT = "3306"
		DB_NAME = "db_app"
		DB_USERNAME = "admin"
		DB_PASSWORD = "rutipass"
	    //JAVA_HOME = "/usr/lib/jvm/jre-1.8.0/"
		NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "https"
        NEXUS_URL = "192.168.3.159:9000"
        NEXUS_REPOSITORY = "backend_main_repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
            
  }
    stages {
	stage('Git') {
	
	stage('Test') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
         dir('.') {
            sh './gradlew clean wrapper --info'
          }
       }
      }
      
    }

  stage('SonarQube analysis') {
    environment {
      SCANNER_HOME = tool 'Sonar-scanner'
    } 
   steps {
       script {
      
           withSonarQubeEnv("SonarQube") {
           sh "$SCANNER_HOME/bin/sonar-scanner  \
           -Dsonar.projectKey=multibranch_backend \
		   -Dsonar.projectVersion=1.0 \
		   -Dsonar.java.binaries=. \
           -Dsonar.sources=. \
           -Dsonar.language=java \
           -Dsonar.host.url=http://192.168.3.159:9000 \
           -Dsonar.login=4a9c38808c74a9fc1efc7fa852f86429141bb0cc"
               }
           }
		  
       }
   }
   
   stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true }
            }
        }
   
     stage('Build') {
      steps {
		sh '''if [ ! -f ./gradlew ]; then \\
                        gradle \\
                            --quiet \\
                            --no-daemon \\
                            --no-build-cache \\
                            wrapper; \\
                        else    ./gradlew \\
                                --quiet \\
                                --no-daemon \\
                                --no-build-cache \\
                                build -x test
                        fi'''
		

	 }
    }
	
	
		stage("Publish to Nexus Repository Manager") {
            steps {
               
				script {
			      dir('.') {
                    def artifact_name = "build"
                    nexusArtifactUploader (
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        repository: NEXUS_REPOSITORY,
                        version: "$BUILD_DATE",
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        groupId: 'devops-training',
                        

						 artifacts: [
                                [artifactId: 'build/libs/backend', file: "build/libs/backend.jar", type: 'jar']
							]
                    )

				  }
				}
			}
		}
   }
   
   post {
        always {
            cleanWs()
        }
        success{
            echo " ---=== SUCCESS ===---"
        }
        failure{
            echo " ---=== FAILURE ===---"
        }
    }
   }
   
   
   


