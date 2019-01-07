node("master") {

	stage('clean workspace'){
		deleteDir()    	 	
	}
	stage('SCM Checkout'){ 
		
		echo "Branch - ${env.BRANCH_NAME}"
		echo "Job name - ${env.JOB_NAME}"
		//temp- to be removed
		def branch = env.BRANCH_NAME
		def job = env.JOB_NAME
		// if the build was triggered by master
		if(branch != null) {
			if("${branch}" =~ /^master$/) {
				buildType = '-RELEASE'
			} else {
				buildType = '-SNAPSHOT'
			}
		} else {
		/*	if(job.endsWith('-master')){
				buildType = '-RELEASE'
			} else {
				buildType = '-SNAPSHOT'
			}
			*/
				buildType = '-RELEASE'
		}
		echo "buildType - ${buildType}"
		
		//Check out from git	
		checkout scm
	}
	stage('Git Commit Hash'){
		sh 'git rev-parse HEAD > commit'
		git_commit = readFile('commit').trim()
		SHORT_GIT_COMMIT = git_commit.take(7)
		echo "Short Git Commit = ${SHORT_GIT_COMMIT}"
		
		
		try {  
 			notifyBuild('STARTED')  
 			echo 'Build Started...'  
 			  
 			currentBuild.displayName = "${env.BUILD_NUMBER + '-' + SHORT_GIT_COMMIT}"  
 			buildVersionNumber = env.BUILD_NUMBER + '-' + SHORT_GIT_COMMIT + buildType  
		} catch (e) {  
 			//If there was an exception thrown, the build failed  
 			currentBuild.result = "FAILED"  
 			throw e  
 		} finally {  
 			//Success or failure, always send notifications  
 			notifyBuild(currentBuild.result)  
 		}	
	
	}
	stage('Cherwell Approval') {
		if("${buildType}" == "-RELEASE") {		
				sh 'git rev-parse HEAD > commit'
				git_commit = readFile('commit').trim()
				SHORT_GIT_COMMIT = git_commit.take(7)
				echo "Short Git Commit = ${SHORT_GIT_COMMIT}"
				def Promotion= SHORT_GIT_COMMIT
				//Promotion= "HelloWorldTest"
				//echo "Short Git Commit = ${SHORT_GIT_COMMIT}"
				cherwellIntegration.validateChange
				{
					promotionID = "${Promotion}"
				}
				echo "CM is Approved. Proceeding with Build"
			}
		else{
			echo "CM Approval not needed for SNAPSHOT build"
		}

	}
	
	 stage('Flyway_migrate')
	 {
	  url = 'jdbc:db2://corde09:3757/DISTDMP'
        flywayrunner commandLineArgs: '-outOfOrder=true -sqlMigrationPrefix=DBR', credentialsId: 'e8fc6a6c-f747-446b-9835-9a68c984fcaa', flywayCommand: 'migrate', installationName: 'flyway', locations:'filesystem:/var/lib/jenkins/workspace/fr-specprod-DISTDMP_master', url: url
        
        }
	
	
	 stage('Flyway_info')
	 {
		
	 url = 'jdbc:db2://corde09:3757/DISTDMP'
        flywayrunner commandLineArgs: '-outOfOrder=true -sqlMigrationPrefix=DBR', credentialsId: 'e8fc6a6c-f747-446b-9835-9a68c984fcaa', flywayCommand: 'info', installationName: 'flyway', locations:'filesystem:/var/lib/jenkins/workspace/fr-specprod-DISTDMP_master', url: url         
        }
	
}
	//Defination for noftify build
		def notifyBuild(String buildStatus = 'STARTED') {
		// build status of null means successful
		buildStatus = buildStatus ?: 'SUCCESS'

		// Default values
		def slackToken = 'anD83B6cGojCah6NUvXpEez8'
		def slackTeam = 'northwesternmutual'
		def colorName = 'RED'
		def colorCode = '#FF0000'
		
		def subject = "${buildStatus}: Job '${env.JOB_NAME} [${SHORT_GIT_COMMIT}]'"
		def summary = "${subject} (${env.BUILD_URL})"
		def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
			<p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

		// Override default values based on build status
		if (buildStatus == 'STARTED') {
			color = 'YELLOW'
			colorCode = '#FFFF00'
		} else if (buildStatus == 'SUCCESS') {
			color = 'GREEN'
			colorCode = '#00FF00'
		} else {
			color = 'RED'
			colorCode = '#FF0000'
	  }

		// Send notifications
		slackSend (color: colorCode, message: summary, token: slackToken, teamDomain: slackTeam)
	}
