node {
  def image = 'runyonsolutions/appinfo'
  def tag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"

  def srcdir = 'github.com/runyontr/canary-app'

   stage('kubectl configuration'){
    withEnv(['PATH+JENKINSHOME=/home/jenkins/bin']) {

        //This assumes there's a kubectl sidecar
        sh """
            curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/linux/amd64/kubectl
            chmod +x kubectl
            mkdir -p /home/jenkins/bin
            mv kubectl /home/jenkins/bin
        """
    }
   }

// Install the desired Go version


  checkout scm

      def root = tool name: 'Go 1.8', type: 'go'
      def workspace = pwd()
  stage('Run Go tests') {
  // Export environment variables pointing to the directory where Go was installed
     withEnv(["GOROOT=${root}", "PATH+GO=${root}/bin"]) {
        sh """
             mkdir -p \$HOME/go/src/${srcdir}
             ln -s ${workspace}/* \$HOME/go/src/${srcdir}/
             cd  \$HOME/go/src/${srcdir}
             export GOPATH=\$HOME/go
             go test ./...
              """
     }
 }


     stage('Build and Push Image') {
      // Export environment variables pointing to the directory where Go was installed
       withEnv(["GOROOT=${root}", "PATH+GO=${root}/bin"]) {

           sh """
             cd  \$HOME/go/src/${srcdir}
             CGO_ENABLED=0 GOOS=linux go build -o app main.go
             cp app \${workspace}/
           """
            docker.withRegistry('https://registry.hub.docker.com', 'Dockerhub') {
                app = docker.build("${image}:${tag}")
                app.push()
            }
       }

     }


     stage("Deploy Application"){
     withEnv(['PATH+JENKINSHOME=/home/jenkins/bin']) {
        //Update the image in the deployment spec
        sh("sed -i 's/IMAGE_TAG/${tag}/g' ./k8s/deployment.yaml")


        switch (env.BRANCH_NAME) {
            // Roll out to canary environment
            case "canary":
                // Change deployed image in canary to the one we just built
                sh("echo Hi this is the canary branch")
                //Change the name of things to be appinfo-canary

                sh("sed -i 's/name: appinfo/name: appinfo-canary/g' ./k8s/deployment.yaml")

                //change the release value to be canary
                sh("sed -i 's#release: stable#release: canary#' ./k8s/deployment.yaml")

                sh("cat ./k8s/deployment.yaml")

                sh("kubectl apply -f k8s/deployment.yaml")

                break

            // Roll out to production
            case "master":
                // Change deployed image in canary to the one we just built
                sh("echo hi master")
                sh("cat ./k8s/deployment.yaml")
                sh("kubectl apply -f k8s/deployment.yaml")
                sh("kubectl apply -f k8s/service.yaml")

                //cleanup the canary build
                sh("kubectl delete deployment appinfo-canary")

                break

            // All other branches shouldn't be deployed
            default:

                sh("echo Not deploying application")
          } //switch
       }//env
     } //stage
} //nodes