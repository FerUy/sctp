pipeline {
	agent any

	tools {
	    jdk 'JDK 11'
		maven 'Maven_3.6.3'
	}

	parameters {
	    string(name: 'SCTP_MAJOR_VERSION', defaultValue: '2.0.2', description: 'The major version number for RestComm SCTP')
	}

	stages{
		stage('Set Version') {
			steps {
				sh "mvn versions:set -DnewVersion=${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}"
				echo "Setting version to ${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER} completed"
			}
		}

		stage("Build") {
			steps {
				echo "Building RestComm SCTP version (#${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER})"
				script {
                    currentBuild.displayName = "#${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}"
                    currentBuild.description = "RestComm SCTP build"
                }
				sh "mvn clean install -DskipTests"

				echo "Maven build completed."
			}
		}

		stage("Release") {
			steps {
				echo "Starting ant build for version #${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}"
 			    dir('release') {
 			        sh "mvn clean install -DskipTests -Prelease -Drelease.name=sctp"
 			    }
			}
		}

		stage('Save Artifacts') {
          steps {
              echo "Archiving RestComm-SCTP-${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}"
              archiveArtifacts artifacts: "release/sctp/", followSymlinks: false, onlyIfSuccessful: true
          }
        }

        stage('Zip Artifacts') {
          steps {
               echo "Archiving RestComm-SCTP-${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}.zip"
               dir('release') {
                    sh "zip -r RestComm-SCTP-${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}.zip sctp/"
               }
          }
        }

        stage('Push to Repo') {
            when {
                anyOf {
                    branch 'master';
                    branch 'release'
                }
            }
            steps {
                sshagent(['ssh_grafana']) {
                    sh "scp release/RestComm-SCTP-${params.SCTP_MAJOR_VERSION}-${BUILD_NUMBER}.zip root@127.0.0.1:/var/www/html/RestComm/sctp/"
                }
            }
        }

        stage('Push to jFrog') {
            when {anyOf {branch 'master'; branch 'release'}}
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }
	}

	post {
		success {
			echo "Successfully build"
		}

		always {
			sh 'rm -rf release/sctp'
			echo "This will be called always. After testing do clean up"
		}
	}
}
