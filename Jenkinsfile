pipeline {

    parameters {
        string(name: 'argocdServer', defaultValue: 'argocd.qa.givelify.com', description: 'Argocd Server')
        string(name: 'applicationName', defaultValue: 'webapp-prod', description: 'Application Name')
        string(name: 'Replicas', defaultValue: '2', description: 'Enter the replicas count')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Enter the branch name')
    }

	environment {
        GIT_USERNAME= 'mohankarki54'
        GITHUB_TOKEN = credentials('mohan-argocd-poc')
        REPO_URL = 'github.com/mohankarki54/helm-demo.git'
        GIT_CREDENTIAL_NAME = "mohan-argocd-poc"
        imagerepo = 'testing'
        imagename = 'nodejs-docker'
        authToken    = credentials('jenkins-argocd-token')
	}

	agent any
	
	stages {
	
    stage('Build Docker Image') {
      steps {
        echo "Build Docker images complete for replica ${params.Replicas}"
        // sh "docker build --no-cache . -t ${imagename}:v${BUILD_NUMBER}"
      }
    }
    
    stage('Tag Docker Image') {
      steps {
        echo "Build Docker images tag complete"
        // sh "docker tag nodejs-docker:v${BUILD_NUMBER} ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
      }
    }
    
    stage('Push Docker Image to Docker Hub') {
      steps {
        echo "Build Docker images push complete"
        // withDockerRegistry([ credentialsId: 'DockerHubCredentials', url: '' ]) {
        //   sh "docker push ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
        // }
      }
    }
    
    stage('Remove Docker Image') {
      steps{
        echo "Remove Docker images locally pushed"
        // sh "docker rmi ${imagename}:v${BUILD_NUMBER}"
        // sh "docker rmi ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
      }
    }
    
    stage('clone and Update Manifest') {
      steps {
        git credentialsId: 'mohan-argocd-poc', url: "https://$REPO_URL", branch: "main"
          dir('webapp1') {
              script {
                sh "sed -i 's/replicas.*/replicas: ${params.Replicas}/g' values-prod.yaml"
                // Check if there are changes to the file
                def gitDiff = sh(script: 'git diff --exit-code values-prod.yaml', returnStatus: true)
                
                if (gitDiff == 0) {
                    echo "No changes to commit,"
                } else {
                    sh "git config user.email mkarki@gmail.com"
                    sh "git config user.name devops-bot"
                    sh "git add values-prod.yaml"
                    sh "git commit -m 'Update replica count to: ${BUILD_NUMBER}'"
                    sh "echo 'updating replicas'"
                    withCredentials([usernamePassword(credentialsId: 'mohan-argocd-poc', passwordVariable: "$GITHUB_TOKEN", usernameVariable: "$GIT_USERNAME")]) {
                        sh "git push https://${GITHUB_TOKEN}@${REPO_URL} HEAD:main -f"
                    }
                }
              }
          }
      }
    }

    stage('Deploy') {
      steps{
        echo "Running Argocd sync command"
        script {
            // Define your Argo CD server URL, application name, and optionally authentication token
            def argocdServer = '$argocdServer'
            def applicationName = '$applicationName'
            def authToken = '$authToken'

            // Construct the command to sync the application
            def command = "argocd"
            def args = ["--server", argocdServer , "app", "sync", applicationName, "--auth-token", authToken]
            
            // Execute the command
            sh "${command} ${args.join(' ')}"
        }
      }
      
	}}

  post { 
    always {
      cleanWs()
    }
  }
  
}
