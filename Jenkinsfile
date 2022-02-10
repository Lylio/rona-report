node {
	def app
	def image = 'registry.hub.docker.com/lylio/coronavirus-tracker'
	def branch = 'main'

	try {
		stage('Clone Repository') {
	    	git branch: branch,
	        	credentialsId: 'Lylio-GitHub',
	        	url: 'https://github.com/Lylio/coronavirus-tracker.git'
	    }

	    stage('Run Unit Tests') {
	        steps {
	            sh './mvnw test'
	        }
	    }

		stage('Build JAR') {
	    	docker.image('maven:3.6.3-jdk-11').inside('-v /root/.m2:/root/.m2') {
	        	sh 'mvn -B clean package'
	        	stash includes: '**/*.jar', name: 'jar'
	    	}
	    }

	    stage('Build Docker Image') {
	    	unstash 'jar'
			app = docker.build image
	    }

	    stage('Push to DockerHub') {
	    	docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_id') {
				app.push("${env.BUILD_NUMBER}")
				app.push("latest")
	        }
	    }
	} catch (e) {
		echo 'Error occurred during build process!'
		echo e.toString()
		currentBuild.result = 'FAILURE'
	} finally {
        junit '**/target/surefire-reports/TEST-*.xml'
	}
}