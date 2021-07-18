pipeline {
 agent any
 environment{
     scannerHome = tool name: 'sonar_scanner_dotnet'
     username = 'aayushgupta01'
     registry = 'aayushgup10/nagpdevopsassignment'
 }
 tools {
  msbuild 'MSBuild'
 }
 stages {
  stage('Checkout') {
   steps {
    git  url: 'https://github.com/Aayush-gupta10/DevopsWebApp.git'
   }
  }
  stage('Start SonarQube Analysis')
  {
    steps{
        withSonarQubeEnv('Test_Sonar') {
            bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-aayushgupta01 -d:sonar.cs.opencover.reportsPaths='XUnitTestProject1/TestResults/*/coverage.*.xml' -d:sonar.cs.xunit.reportsPaths='XUnitTestProject1/TestResults/devopsassignmenttestoutput.xml'"
        }
    }
      
  }
  stage('Restore PACKAGES') {
   steps {
    bat "dotnet restore"
   }
  }
  stage('Clean') {
   steps {
    bat 'dotnet clean'
   }
  }
  stage('Build') {
   steps {
    bat 'dotnet build -c Release -o DevopsWebApp/app/build'
   }
 }
 stage('Test: Unit Test'){
   steps {
     bat 'dotnet test XUnitTestProject1/XUnitTestProject1.csproj --collect="XPlat Code Coverage" -l:trx;LogFileName=devopsassignmenttestoutput.xml'
     }
  }
 stage('Stop SonarQube analysis') {
    steps{
        withSonarQubeEnv('Test_Sonar') {
            bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
        }
    }
 }
  stage('Create docker image')
  {
      steps{
           bat "docker build -t i-${username}-master ."
      }
     
  }
  stage('Push image to docker hub') {
      steps {
          bat "docker tag i-${username}-master ${registry}:$BUILD_NUMBER"
            withDockerRegistry(credentialsId: 'DockerHub', url: '') {
                bat "docker push ${registry}:$BUILD_NUMBER"
        }
      }
    }
    stage('Docker deployment')
    {
        steps{
            bat "docker run --name c-${username}-master -d -p 7100:80 ${registry}:$BUILD_NUMBER"
        }
    }
  }  
 }