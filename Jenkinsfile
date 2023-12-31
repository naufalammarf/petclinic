def appName = 'petclinic-user102'
def gitUrl = 'https://gitlab.com/Naufalamm/petclinic-user102.git'
def branch = 'master'
def imageRegistry= 'demowil.jfrog.io/docker-dev'
def namespace = 'petclinic'


node ('master'){
	stage ("Prepare") {
		cleanWs deleteDirs: true
		checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[credentialsId: 'gitlab-user102', url: "${gitUrl}"]], branches: [[name: "${branch}"]]]

	}

	stage ("Build") {
        sh """
        /opt/apache-maven-3.9.4/bin/mvn clean install -Dmaven.test.skip=true
		"""
	}

	stage ("Create Image") {
		withCredentials([usernamePassword(credentialsId: 'image-registry', passwordVariable: 'regPassword', usernameVariable: 'regUsername')]) {
        	sh """
            sudo /usr/bin/docker login ${imageRegistry} -u ${regUsername} -p ${regPassword}
			sudo /usr/bin/docker build -t ${imageRegistry}/${appName}:$BUILD_NUMBER .
			sudo /usr/bin/docker push ${imageRegistry}/${appName}:$BUILD_NUMBER
			sudo /usr/bin/docker rmi ${imageRegistry}/${appName}:$BUILD_NUMBER
			"""
		}
	}

	stage ("Deploy Application") {

		sh """
		sed -i 's|<<IMAGETAG>>|${imageRegistry}/${appName}:$BUILD_NUMBER|g' kubernetes/deployment.yaml
		sed -i 's|<<APPNAME>>|${appName}|g' kubernetes/deployment.yaml
		sed -i 's|<<APPNAME>>|${appName}|g' kubernetes/service.yaml
		kubectl apply -f kubernetes/deployment.yaml -n ${namespace} #--kubeconfig /var/lib/jenkins/.kube/configk8s
		kubectl apply -f kubernetes/service.yaml -n ${namespace}
		kubectl get svc -n petclinic | grep ${appName}
		"""
	}
	
	stage ("Access Port") {
		sh """
		kubectl get svc -n petclinic | grep ${appName}
		echo "YOUR APPLICATION PORT:"
		kubectl get svc -n petclinic -o wide | grep ${appName}
		"""
		
	}
}
