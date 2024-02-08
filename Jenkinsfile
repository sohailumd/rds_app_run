pipeline {
    agent any
    
    environment {
        PG_HOST = 'demo-postgresql.c4fcwcpo9bzm.us-east-2.rds.amazonaws.com'
        PG_PORT = '5432'
        PG_DATABASE = 'postgres'
        PG_pg = credentials('pgdbcreds')
    }
    
    stages {
        stage('DML-DDL-Run') {
            steps {
                script {
                    echo 'Cloning repository...'
                    sh '''
                    git --version
                    which git
                    whoami
                    ls -l
                    '''
					def sqlQuery1 = readFile('psql_scripts/Table_Create.sql')
					def sqlQuery2 = readFile('psql_scripts/Table_Insert.sql')
                    sh """
                    pwd
                    ls -l
					PGPASSWORD=${PG_pg_PSW} psql -h ${env.PG_HOST} -p ${env.PG_PORT} -d ${env.PG_DATABASE} -U ${PG_pg_USR} -c \"${sqlQuery1}\"
					PGPASSWORD=${PG_pg_PSW} psql -h ${env.PG_HOST} -p ${env.PG_PORT} -d ${env.PG_DATABASE} -U ${PG_pg_USR} -c \"${sqlQuery2}\"
					echo "HURRAY"
					"""
                }
            }
        }
        stage('Build') {
            steps {
                    checkout([$class: 'GitSCM', 
                              branches: [[name: 'main']], 
                              doGenerateSubmoduleConfigurations: false, 
                              extensions: [], 
                              userRemoteConfigs: [[url: 'git@github.com:sohailumd/rds_app_run.git']]])
                sh ' pwd; cd demo-app; ls -l; chmod +x gradlew; cd demo-app'
                echo 'Running build automation'
                sh 'sudo gradlew build --no-daemon'
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