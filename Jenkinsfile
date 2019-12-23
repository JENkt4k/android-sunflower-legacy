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
              echo currentBuild.result
            }       
          }
        }
        stage('test'){
          steps{
            script {
              try {
                sh './gradlew test'
              } catch (Exception e) {
                echo e.getMessage()
                echo "test failed"
              }
              echo currentBuild.result
            }       
          }   
        }
      }
    }
    stage('junit'){
      steps{
        script {
          try {
            junit 'app/build/reports/**/*.xml'
          } catch (Exception e) {
            echo e.getMessage()
            echo "junit failed"
          }
          echo currentBuild.result
        }       
      }
    }
    stage('coverage'){
      steps{
        script {
          try {
            sh './gradlew -Pcoverage=true createDebugCoverageReport'
            //sh './gradlew -Pcoverage=true testDebugUnitTest' 
          } catch (Exception e) {
            echo e.getMessage()
            echo "coverage failed"
          }
          echo currentBuild.result
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
      archiveArtifacts 'app/build/reports/**/*.xml'
      androidLint canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'app/build/reports/**/*', unHealthy: ''
      archiveArtifacts 'app/build/outputs/apk/**/*.apk'
      archiveArtifacts 'app/**/*test*/**/*.xml'
      archiveArtifacts 'app/**/*test*/**/*.js'
      archiveArtifacts 'app/**/*test*/**/*.html'
      dir('app/build/test-results'){ 
        deleteDir()
      }
      
    }
  }
} 