node('maven') {
  stage('Build') {
    git url: "https://github.com/siamaksade/cart-service.git"
    sh "mvn package"
  }
  stage('Test') {
    parallel(
      "Cart Tests": {
        sh "mvn verify -P cart-tests"
      },
      "Discount Tests": {
        sh "mvn verify -P discount-tests"
      }
    )
  }
  stage('Build Image') {
    sh "oc start-build cart --from-file=target/cart.jar --follow"
  }
  stage('Deploy') {
    openshiftDeploy depCfg: 'cart'
    openshiftVerifyDeployment depCfg: 'cart', replicaCount: 1, verifyReplicaCount: true
  }
  stage('System Test') {
    sh "curl -s http://cart:8080/api/cart/FOO | grep 'cartItemTotal'"
  }
}