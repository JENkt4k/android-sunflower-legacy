pipeline {
  agent { 
    docker {
       image 'circleci/android:api-29-node' 
    }
  }
  stages {
    stage('clean') {
      steps {
          sh './gradlew clean'  
      }
    }
    stage('build') {
      steps {
          sh './gradlew build'
      }
    }
    stage('Lint and Unit Test'){
      parallel {
        stage('lint'){
          steps{
            script {
              try {
                sh './gradlew lint'
              } catch (Exception e) {
                echo e.getMessage()
                echo "Lint failed"
              }
            }       
          }
        }
        stage('test'){
          steps{
            script {
              try {
                sh './gradlew test'
                junit 'app/build/test-results/**/*.xml'
              } catch (Exception e) {
                echo e.getMessage()
                echo "test failed"
              }
            }       
          }   
        }
      }
    }
    stage('Sonarqube') {
      environment {
        scannerHome = tool 'SonarQubeScanner'
      }
      steps {
        withSonarQubeEnv('sonarqube') {
          sh "${scannerHome}/bin/sonar-scanner"
        }
        timeout(time: 30, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('createAPK'){
      steps{
        sh './gradlew assembleRelease'  
      }
    }
  }
  post {
    always {
      androidLint canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'app/build/reports/**/*', unHealthy: ''
      archiveArtifacts 'app/build/outputs/apk/**/*.apk'
      archiveArtifacts 'app/build/**/tests/**/*.xml'
      archiveArtifacts 'app/build/**/tests/**/*.js'
      archiveArtifacts 'app/build/**/tests/**/*.html'
      dir('app/build/test-results'){ 
        deleteDir()
      }
      
    }
  }
} 