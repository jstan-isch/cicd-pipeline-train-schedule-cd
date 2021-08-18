pipeline {
    agent any
    parameters {
    string(name: 'PERSON', defaultValue: 'jstan-isch', description: 'Running build automation on jenkins server')
    }

    stages {
        stage('build') {
            steps {
                echo "${params.PERSON}"
                echo 'Running build automation'
                sh './gradlew build'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip',
                    allowEmptyArchive: true
                fingerprint targets: 'dist/trainSchedule.zip'
            }
        }

        stage('staging'){
            when {

                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'staging', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    sshPublisher(
                        alwaysPublishFromMaster: true,
                        failOnError: true
                        continueOnError: false,
                        publishers: [
                            configName: 'staging',
                            sshCredential: [
                                username: '$USERNAME',
                                encryptedPassphrase: '$USERPASS'
                            ],
                            transfers: [
                                sourceFiles: 'dist/trainSchedule.zip',
                                removePrefix: 'dist/',
                                removeDirectory: '/tmp',
                                execCommand: 'sudo /usr/bin/systemctl stop train-schedule \
                                 && rm -rf /opt/train-schedule/* unzip trainSchedule.zip -d /opt/train-schedule \
                                 && /usr/bin/systemctl start train-schedule'
                            ]
                        ]
                    )
                }
            }
        }

        stage('deploy to staging server'){
            when {

                branch 'master'
            }
            steps {
                input 'Deploy to production server?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'staging', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
                    sshPublisher(
                        alwaysPublishFromMaster: true,
                        failOnError: true
                        continueOnError: false,
                        publishers: [
                            configName: 'staging',
                            sshCredential: [
                                username: '$USERNAME',
                                encryptedPassphrase: '$USERPASS'
                            ],
                            transfers: [
                                sourceFiles: 'dist/trainSchedule.zip',
                                removePrefix: 'dist/',
                                removeDirectory: '/tmp',
                                execCommand: 'sudo /usr/bin/systemctl stop train-schedule \
                                 && rm -rf /opt/train-schedule/* unzip trainSchedule.zip -d /opt/train-schedule \
                                 && /usr/bin/systemctl start train-schedule'
                            ]
                        ]
                    )
                }
            }
        }
    }
}
