node('maven') {
  stage('Build App') {
    git url: "https://github.com/siamaksade/cart-service.git"
    sh "mvn clean package -s src/main/config/settings.xml"
  }
  stage('Integration Test') {
    sh "mvn verify -s src/main/config/settings.xml"
  }
  stage('Build Image') {
    sh "oc start-build cart --from-file=target/cart.jar --follow"
  }
  stage('Deploy') {
    openshiftDeploy depCfg: 'cart'
    openshiftVerifyDeployment depCfg: 'cart', replicaCount: 1, verifyReplicaCount: true
  }
  stage('Component Test') {
    sh "curl -s -X POST http://cart.dev.svc.cluster.local:8080/api/cart/dummy/666/1"
    sh "curl -s http://cart.dev.svc.cluster.local:8080/api/cart/dummy | grep 'Dummy Product'"
  }

  def tag="blue"
  def altTag="green"
  
  stage('Promote') {
    sh "oc get route cart -n prod -o jsonpath='{ .spec.to.name }' > activeservice"
    activeService = readFile('activeservice').trim()
    if (activeService == "cart-blue") {
      tag = "green"
      altTag = "blue"
    }
    openshiftTag sourceStream: 'cart', sourceTag: 'latest', namespace: 'dev', destinationStream: 'cart', destinationTag: "prod-${tag}", destinationNamespace: 'prod'
    openshiftVerifyDeployment deploymentConfig: "cart-${tag}", replicaCount: '1', verifyReplicaCount: true, namespace: 'prod'
  }

  stage('End-To-End Test') {
    sh "curl -s -X POST http://cart-${tag}.prod.svc.cluster.local:8080/api/cart/dummy/444434/1"
    sh "curl -s http://cart-${tag}.prod.svc.cluster.local:8080/api/cart/dummy | grep 'Pebble Smart Watch'"
  }

  stage('Approve Go Live') {
    timeout(time:15, unit:'MINUTES') {
      input message:'Go Live in Prod?'
    }
  }

  stage('Go Live') {
    sh "oc set route-backends cart cart-${tag}=100 cart-${altTag}=0 -n prod"
  }
}
