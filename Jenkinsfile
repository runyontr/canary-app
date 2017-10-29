node {
  def image = runyonsolutions/appinfo
  def tag = ${env.BRANCH_NAME}-${env.BUILD_NUMBER}



  checkout scm

  stage 'Build image'
  sh("CGO_ENABLED=0 GOOS=linux go build -o app main.go"
  sh("docker build -t ${image}:${tag} .")

  stage 'Run Go tests'
  sh("go test ./...")

  stage 'Push image to registry'
  sh("docker push ${image}:${tag}")

  stage "Deploy Application"
  //Update the image in the deployment spec
  sh("sed -i.bak 's#${image}#${image}:${tag}#' ./k8s/deployment.yaml")


  switch (env.BRANCH_NAME) {
    // Roll out to canary environment
    case "canary":
        // Change deployed image in canary to the one we just built
        sh("echo Hi this is the canary branch")
        //Change the name of things to be appinfo-canary

        sh("sed -i.bak 's#name:  appinfo#name:  appinfo-canary#s")



        //change the release value to be canary
        sh("sed -i.bak 's#release:  stable#release:  canary#' ./k8s/deployment.yaml")

        sh("echo ./k8s/deployment.yaml")



//        sh("kubectl --namespace=production apply -f k8s/services/")
//        sh("kubectl --namespace=production apply -f k8s/canary/")
//        sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break

    // Roll out to production
    case "master":
        // Change deployed image in canary to the one we just built
        sh("echo hi master")
        sh("echo ./k8s/deployment.yaml")

        break

    // All other branches shouldn't be deployed
    default:

        sh("echo 'Not deploying application")
  }
}