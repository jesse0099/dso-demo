pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }

  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }

    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }

        stage('SCA') {
          options {
            retry(2)
            timeout(time: 25, unit: 'MINUTES')
          }
          steps {
            container('maven') {
              withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                  sh '''
                    mvn -B -ntp org.owasp:dependency-check-maven:10.0.3:check \
                      -DnvdApiKey=$NVD_API_KEY \
                      -DfailOnError=false
                  '''
                }
              }
            }
          }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true
            }
          }
        }

        stage('GenerateSBOM') {
          steps {
            container('maven') {
              sh'mvnorg.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
            }
          } 
          post {
            success {
              dependencyTrackPublisherprojectName:'sample-spring-app',projectVersion:'0.0.1',artifact:'target/bom.xml',autoCreateProjects:true,synchronous:truearchiveArtifactsallowEmptyArchive:true,artifacts:'target/bom.xml',fingerprint:true,onlyIfSuccessful:true
            }
          }
        }

        stage('OSS LicenseChecker') {
          steps {
            container('licensefinder') {
              sh 'ls -al'
              sh '''#!/bin/bash --login
                      /bin/bash --login
                      rvm use default
                      gem install license_finder
                      license_finder
                   '''
            }
          }
        }
      }
    }

    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }

        stage('OCI Image BnP') {
          steps {
            container('kaniko') {
              sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/jese69/dso-demo --force'
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        // TODO
        sh 'echo done'
      }
    }
  }
}
