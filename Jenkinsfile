
node {
	
    stage('Clone sources') {
        git url: 'https://github.com/danielperga7/movie-analyst-ui.git'
    }
    stage ('build') {
   	sh 'npm install'
    }
    stage ('test') {
    	sh 'npm test'
    }
	stage ('deploy'){
    withEnv(["GOOGLE_PROJECT_ID = ramp-up-247818","HOME=."]){
    withCredentials([file(credentialsId: 'JENKINS_SERVICE_ACCOUNT', variable: 'GOOGLE_SERVICE_ACCOUNT_KEY')]) {
         sh 'echo ------------------setting up google cloud ------------------'

                 sh """
        	        #!/bin/bash
        	        curl -o /tmp/google-cloud-sdk.tar.gz https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-231.0.0-linux-x86_64.tar.gz;
                    tar -xvf /tmp/google-cloud-sdk.tar.gz -C /home/jenkins/;
		            /home/jenkins/google-cloud-sdk/install.sh -q;
                    source /home/jenkins/google-cloud-sdk/path.bash.inc;
			        gcloud config set project ${GOOGLE_PROJECT_ID};
			        gcloud components install kubectl;
                    PATH=$PATH:/home/jenkins/google-cloud-sdk/bin
                    source /home/jenkins/google-cloud-sdk/path.bash.inc;
                    [[ ":$PATH:" != *":/home/jenkins/google-cloud-sdk/bin:"* ]] && PATH="/home/jenkins/google-cloud-sdk/bin:${PATH}"
                    echo $PATH
			        gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY};
                    gcloud components update
                    """
                sh 'echo -------------------Account configured ------------------'
                sh '/usr/bin/curl -o /tmp/front-dockerfile/dockerfile https://raw.githubusercontent.com/Danielperga97/myDevopsRampUp/develop/containers/frontend/dockerfile'
                sh "docker build --no-cache -t gcr.io/ramp-up-247818/movie-analyst-ui:${env.BUILD_NUMBER} /tmp/front-dockerfile/"
                sh "docker tag gcr.io/ramp-up-247818/movie-analyst-ui:${BUILD_NUMBER} gcr.io/ramp-up-247818/movie-analyst-ui:latest"
                sh "gcloud docker -- push  gcr.io/ramp-up-247818/movie-analyst-ui:${BUILD_NUMBER}"
                sh 'gcloud docker -- push  gcr.io/ramp-up-247818/movie-analyst-ui:latest'
                sh  '''   
                #!/bin/bash 
                docker system prune -a -f;
                gcloud container clusters get-credentials gke-cluster-1 --zone us-east1-b;
                /home/jenkins/google-cloud-sdk/bin/kubectl set image deployment.apps/movie-analyst-ui movie-analyst-ui=gcr.io/ramp-up-247818/movie-analyst-ui:${BUILD_NUMBER}
                '''
    }}
    }
    
}


