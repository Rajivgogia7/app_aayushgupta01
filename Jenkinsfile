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
                    bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-aayushgupta01 -d:sonar.cs.opencover.reportsPaths='XUnitTestProject1/TestResults/*/coverage.cobertura.xml' -d:sonar.cs.xunit.reportsPaths='XUnitTestProject1/TestResults/devopsassignmenttestoutput.xml'"
                    //bat "${scannerHome}/SonarScanner.MSBuild.exe begin /k:sonar-aayushgupta01 /d:sonar.cs.opencover.reportsPaths=XUnitTestProject1/coverage.opencover.xml /d:sonar.coverage.exclusions='**Test*.cs'"
           
                }
            }

        }
        stage('Build') {
            steps {
                bat 'dotnet clean'
                bat 'dotnet build -c Release -o DevopsWebApp/app/build'
                //bat 'dotnet test XUnitTestProject1/XUnitTestProject1.csproj --collect="XPlat Code Coverage" -l:trx;LogFileName=devopsassignmenttestoutput.xml'
                bat 'dotnet test XUnitTestProject1\\XUnitTestProject1.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=devopsassignmenttestoutput.xml'
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
        stage('Release artifact') {
            when {
                branch 'develop'
            }
            steps {
                bat 'dotnet publish -c Release'
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
                    "Precontainer Check": {
                        script {
                            if (env.BRANCH_NAME == 'master') {
                                env.port = 7200
                            } else {
                                env.port = 7300
                            }
                            env.containerId = bat(script: "docker ps -f publish=${port} -q", returnStdout: true).trim().readLines().drop(1).join('')
                            if (env.containerId != '') {
                                echo "Stopping and removing container running on ${port}"
                                bat "docker stop $env.containerId"
                                bat "docker rm $env.containerId"
                            } else {
                                echo "No container running on ${port}  port."
                            }
                        }
                    },
                    PushtoDockerHub: {
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
                    bat "docker run --name c-${username}-$env.BRANCH_NAME -d -p  ${port}:80 ${registry}:latest"
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
