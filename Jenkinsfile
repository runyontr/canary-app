node {
  def image = 'runyonsolutions/appinfo'
  def tag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"

  def srcdir = 'github.com/runyontr/canary-app'


// Install the desired Go version
  def root = tool name: 'Go 1.8', type: 'go'

  checkout scm

  stage 'Run Go tests' {
   // Export environment variables pointing to the directory where Go was installed
      withEnv(["GOROOT=${root}", "PATH+GO=${root}/bin"]) {
         sh """
              go version
              go env
              mkdir -p \$GOPATH/src/${srcdir}
              ln -s \$(realpath .) \$GOPATH/src/${srcdir}
              cd  \$GOPATH/src/${srcdir}
              go test ./...
               """
      }


  }



  stage 'Build image' {
   // Export environment variables pointing to the directory where Go was installed
    withEnv(["GOROOT=${root}", "PATH+GO=${root}/bin"]) {

        sh """
           go version
           go env
          cd  \$GOPATH/src/${srcdir}
          CGO_ENABLED=0 GOOS=linux go build -o app main.go
          docker build -t ${image}:${tag} .
        """
    }

  }

  




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