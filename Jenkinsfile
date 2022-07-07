pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build '
        archiveArtifacts 'dist/trainSchedule.zip'
      }
    }

    stage('DeployToStaging') {
      when {
        branch 'master'
      }
      steps {
        withCredentials(bindings: [usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          sshPublisher(failOnError: true, publishers: [
                                                    sshPublisherDesc(
                                                            configName: 'staging',
                                                            sshCredentials: [
                                                                    username: "$USERNAME",
                                                                    encryptedPassphrase: "$USERPASS"
                                                                ], 
                                                                transfers: [
                                                                        sshTransfer(
                                                                                sourceFiles: 'dist/trainSchedule.zip',
                                                                                removePrefix: 'dist/',
                                                                                remoteDirectory: '/tmp',
                                                                                execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                                                            )
                                                                        ]
                                                                    )
                                                                ])
                  }

                }
              }

              stage('DeployToProduction') {
                when {
                  branch 'master'
                }
                steps {
                  input 'Does the staging environment look OK?'
                  milestone 1
                  withCredentials(bindings: [usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(failOnError: true, publishers: [
                                                                        sshPublisherDesc(
                                                                                configName: 'production',
                                                                                sshCredentials: [
                                                                                        username: "$USERNAME",
                                                                                        encryptedPassphrase: "$USERPASS"
                                                                                    ], 
                                                                                    transfers: [
                                                                                            sshTransfer(
                                                                                                    sourceFiles: 'dist/trainSchedule.zip',
                                                                                                    removePrefix: 'dist/',
                                                                                                    remoteDirectory: '/tmp',
                                                                                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                                                                                )
                                                                                            ]
                                                                                        )
                                                                                    ])
                            }

                          }
                        }

                      }
                    }