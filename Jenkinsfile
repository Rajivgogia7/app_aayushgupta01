pipeline {
    agent any
    environment {
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
                git url: 'https://github.com/Aayush-gupta10/app_aayushgupta01.git'
            }
        }
        stage('nuget restore') {
            steps {
                bat "dotnet restore"
            }
        }
        stage('start sonarqube analysis') {
            when {
                branch 'master'
            }
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-aayushgupta01 -d:sonar.cs.opencover.reportsPaths='XUnitTestProject1/TestResults/*/coverage.*.xml' -d:sonar.cs.xunit.reportsPaths='XUnitTestProject1/TestResults/devopsassignmenttestoutput.xml'"
                }
            }

        }
        stage('Build') {
            steps {
                bat 'dotnet clean'
                bat 'dotnet build -c Release -o DevopsWebApp/app/build'
                bat 'dotnet test XUnitTestProject1/XUnitTestProject1.csproj --collect="XPlat Code Coverage" -l:trx;LogFileName=devopsassignmenttestoutput.xml'
            }
        }
        stage('stop sonarqube analysis') {
            when {
                branch 'master'
            }
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
                }
            }
        }
        stage('Docker image') {
            steps {
                bat "docker build -t i-${username}-$env.BRANCH_NAME ."
                bat "docker tag i-${username}-$env.BRANCH_NAME ${registry}:$BUILD_NUMBER"
                bat "docker tag i-${username}-$env.BRANCH_NAME ${registry}:latest"
            }

        }
        stage('Containers') {
            steps {
                parallel(
                    PrecontainerCheck:{
                        bat'''
                        ffor /f %%i in ('docker ps -f "publish=7300" -q') do set containerId=%%i
                        echo %containerId%
                        If NOT "%containerId%" == "" (
					docker stop %containerId%
					docker rm -f %containerId%
				)
				ELSE(
					echo "Container is not running on 7300"
				)
                        '''
                    },
                    PushtoDockerHub:{
                        withDockerRegistry(credentialsId: 'DockerHub', url: '') {
                            bat "docker push ${registry}:$BUILD_NUMBER"
                            bat "docker push ${registry}:latest"
                        }
                    }
                )
            }
        }
        stage('Docker deployment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'master') {
                        bat "docker run --name c-${username}-$env.BRANCH_NAME -d -p 7200:80 ${registry}:latest"
                    } else if (env.BRANCH_NAME == 'develop') {
                        bat "docker run --name c-${username}-$env.BRANCH_NAME -d -p 7300:80 ${registry}:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                bat "kubectl apply -f deployment.yaml"
            }
        }
    }
}
