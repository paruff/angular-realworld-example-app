def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
 containerTemplate(name: 'node', image: 'node:8.15.1-jessie-slim', ttyEnabled: true, command: 'cat'),
  containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'gradle', image: 'gradle:5.2.1-jdk8', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
volumes: [
    hostPathVolume(mountPath: '/root/.m2/repository', hostPath: '/root/.m2/repository'),
  hostPathVolume(mountPath: '/home/jenkins/.m2', hostPath: '/home/jenkins/.m2'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
        
        stage('Get the project') {
            checkout scm
            container('node') {

                stage('Validate project') {
                    sh 'echo  "validate"'
                }
                
                stage('Load modules') {
                    sh 'yarn install'
                }
                
                stage('Test') {
                    sh '''
                      echo "$(npm bin)/ng test --progress=false --watch false"
                      echo "test me"
                    '''
                  //junit "test-results.xml"
                }
                
// TODO
//  sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target               
//                stage('Scan components Maven project') {
//                    sh 'mvn -B -Djavax.net.ssl.trustStore=/path/to/cacerts dependency-check:check'
//                }
            
                stage 'Package and Code Analysis'
                    withSonarQubeEnv {
                        sh '$(npm bin)/ng lint'
                        sh 'sonar-scanner  -Dsonar.projectKey=angular-conduit-ui -Dsonar.sources=.' 
                    }

                    stage('SonarQube analysis') {
    // requires SonarQube Scanner 2.8+
    def scannerHome = tool 'SonarQubeScanner';
    withSonarQubeEnv('My SonarQube Server') {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }
                
                stage('Build') {
                    sh '$(npm bin)/ng build --prod --build-optimizer'
                } 
                
            }
        }
        stage('Create Docker images') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            rm -rf node_modules
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            docker build -t paruff/realworld:${gitCommit} .
            docker push paruff/realworld:${gitCommit}
            """
        }
      }
    }

    }
}
