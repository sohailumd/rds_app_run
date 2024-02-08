pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh ' pwd; ls -l; sudo rm -rf dist; ls -l; chmod +x gradlew'
                    checkout([$class: 'GitSCM', 
                              branches: [[name: 'main']], 
                              doGenerateSubmoduleConfigurations: false, 
                              extensions: [], 
                              userRemoteConfigs: [[url: 'git@github.com:sohailumd/demo-app.git']]])
                sh ' pwd; ls -l; chmod +x gradlew'
                echo 'Running build automation'
                sh 'sudo ./gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('deploy-app') {
            steps {
                script {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'demo-app-server',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'rm -rf /home/demouser/demo-app/*; unzip /tmp/trainSchedule.zip -d /home/demouser/demo-app/ '
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}