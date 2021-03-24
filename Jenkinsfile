pipeline {
    agent any
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "MVN3"
        jdk "JDK8"
    }
    stages {
        stage('PullSourcecode') {
            steps {
                // Get some code from a GitHub repository
                git credentialsId: 'github', url: 'git@github.com:sathishbob/jenkins_test.git'
                }
            }
        stage('build application') {
              steps {
                  // Run Maven on a Unix agent.
                  sh "mvn -Dmaven.test.failure.ignore=true -f api-gateway clean package"
              }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit 'api-gateway/target/surefire-reports/*.xml'
                    archiveArtifacts 'api-gateway/target/*.jar'
                }
            }
        }
              stage('sonarqube analysis') {
                  steps {
                      script {
                          scannerHome = tool 'sonar';
                          withSonarQubeEnv('sonar') {
                              sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectName=javaapplication -Dsonar.projectKey=java:findbugs -Dsonar.projectVersion=1.0 -Dsonar.projectBaseDir=$WORKSPACE -Dsonar.sources=$WORKSPACE -Dsonar.java.libraries=$WORKSPACE -Dsonar.java.binaries=$WORKSPACE"
                              sh "sleep 60"
                          }
                      }
                  }
              }
              stage('qualityGate') {
                    steps {
                        script {
                            timeout(time: 1, unit: 'HOURS') {
                                waitForQualityGate abortPipeline: true
                            }
                        }
                    }
               }
    }
}
