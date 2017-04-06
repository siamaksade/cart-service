node('maven') {
  stage('Build Service') {
    git url: "https://github.com/siamaksade/cart-service.git"
    sh "mvn package"
  }
  stage('Build Image') {
    sh "oc start-build cart --from-file=target/cart.jar --follow"
  }
  stage('Deploy Image') {
    openshiftDeploy depCfg: 'cart'
    openshiftVerifyDeployment depCfg: 'cart', replicaCount: 1, verifyReplicaCount: true
  }
  stage('Test') {
    sh "curl -s http://cart:8080/api/cart/FOO | grep 'cartItemTotal'"
  }
}