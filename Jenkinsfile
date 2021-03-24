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
                git credentialsId: 'github', url: 'git@github.com:kavyapendyala/jenkins_test.git'
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
        stage('DependencyCheck') {
              steps {
                  script {
                      sh '''DC_VERSION="latest"
DC_DIRECTORY=$HOME/OWASP-Dependency-Check
DC_PROJECT="dependency-check scan: $(pwd)"
DATA_DIRECTORY="$DC_DIRECTORY/data"
CACHE_DIRECTORY="$DC_DIRECTORY/data/cache"
if [ ! -d "$DATA_DIRECTORY" ]; then
    echo "Initially creating persistent directory: $DATA_DIRECTORY"
    mkdir -p "$DATA_DIRECTORY"
fi
if [ ! -d "$CACHE_DIRECTORY" ]; then
    echo "Initially creating persistent directory: $CACHE_DIRECTORY"
    mkdir -p "$CACHE_DIRECTORY"
fi
# Make sure we are using the latest version
docker pull owasp/dependency-check:$DC_VERSION
docker run --rm \\
    -e user=$USER \\
    -u $(id -u ${USER}):$(id -g ${USER}) \\
    --volume $(pwd):/src:z \\
    --volume "$DATA_DIRECTORY":/usr/share/dependency-check/data:z \\
    --volume $(pwd)/odc-reports:/report:z \\
    owasp/dependency-check:$DC_VERSION \\
    --scan /src \\
    --format "ALL" \\
    --project "$DC_PROJECT" \\
    --out /report'''
                  }
              }
              }
    }
}
