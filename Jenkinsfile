pipeline{
    agent {
        label 'buildAgent'
    }
    tools {
        maven 'maven'
    }
    stages{
        stage('checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/devops911/PruebaTecnica.git'
            }
        }
        stage('build'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }
        stage('sonar scan'){
            steps{
                script{
                    withSonarQubeEnv('sonarqube') {
                        def sonarqubeScannerHome = tool name: 'sonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh "${sonarqubeScannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                    }
                }
            }
        }
        stage('docker build'){
            steps{
                script{
                    docker.withRegistry( '', 'dockerHubCreds' ) {
                        registry = "xle2911/microservices"
                        dockerImage = docker.build registry + ":$BUILD_NUMBER"
                        dockerImage.push()
                        dockerImage = docker.build registry
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deployment'){
            agent {
                label 'deploymentAgent'
            }
            steps{
                script {
                    dockerContainerStatus = sh (
                        script: "docker stop micronservices && docker rm micronservices",
                        returnStatus: true
                    )
                    sh "docker run -d --name micronservices -p 8080:8080 xle2911/microservices"
                }
            }
        }
    }
}
