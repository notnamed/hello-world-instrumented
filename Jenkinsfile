#!groovy

def project = 'jordan_cooks'
def appName = 'hello-world-instrumented'
def feSvcName = "${appName}"
def namespace = 'monitoring-demo'
def imageTag = "quay.io/${project}/${appName}:canary.v${env.BUILD_NUMBER}"

node {
  checkout scm
  sh("printenv")
	
  stage 'Login to Quay.io'
  sh("docker login -u=\"${env.quay_username}\" -p=\"${env.quay_password}\" quay.io")
	
  stage 'Build image'
  sh("docker build -t ${imageTag} .")

//  stage 'Run Go tests '
//  sh("docker run ${imageTag} go test")

  stage 'Push image to Quay.io registry'
  sh("docker push ${imageTag}")

  stage "Deploy Canary"
//  switch (env.BRANCH_NAME) {
//     case "master":  
         // Roll out to canary environment
         // Change deployed image in canary to the one we just built
         sh("sed -i.bak 's#quay.io/${project}/${appName}:.*\$#${imageTag}#' ./k8s/canary/*.yaml")
//	 sh("sed -i.bak 's#python-api-canary#python-api-v${env.BUILD_NUMBER}#' ./k8s/canary/*.yaml")
         sh("kubectl --namespace=${namespace} apply -f k8s/services/")
         sh("kubectl --namespace=${namespace} apply -f k8s/canary/")
//    break
//    default:
//    break
//  }
}
stage 'Verify Canary'
def didTimeout = false
def userInput = true
try {
  timeout(time:5, unit:'DAYS') {
      userInput = input(id: 'promoteToProd', message: 'Approve rollout to production?')
      echo "userInput: [${userInput}]" 
  }
} catch(err) { // timeout reached or input false
    echo "Rollout Aborted"
    echo "userInput: [${userInput}]"
    currentBuild.result = 'FAILURE'
    error('Aborted')
}

stage 'Rollout to Production'
node{ 
  checkout scm 
  // Roll out to production environment
  // Change deployed image in canary to the one we just built
  sh("sed -i.bak 's#quay.io/${project}/${appName}:.*\$#${imageTag}#' ./k8s/production/*.yaml")
  //sh("kubectl --namespace=${namespace} apply -f k8s/services/")
  sh("kubectl --namespace=${namespace} apply -f k8s/production/")
  currentBuild.result = 'SUCCESS'
}
