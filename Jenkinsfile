pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    evironment {
        appVersion = ''
        nexusUrl = 'http://localhost:8081'
    }
    
    stages {
        stage('read the version') {
            steps {
               script {
                   def version = readJSON file: 'package.json'
                   appVersion = version.version
                   echo "Version is ${appVersion}"
               }
            }
        }
        stage('build') {
            steps {
               sh """
                zip -q -r frontend.${appVersion}.zip * -x Jenkinsfile -x frontend.${appVersion}.zip
                ls -ltr              
               """
            }
        }

        stage('Upload to nexus') {
            steps {
               sh """
                curl -v -u admin:admin123 --upload-file frontend.${appVersion}.zip http://localhost:8081/repository/expense-frontend/frontend.${appVersion}.zip
               """
            }
        }
        stage('Nexus artifact uploader') {
            steps {
               script {
                   nexusArtifactUploader(
                        NexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '${nexusUrl}',
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: 'frontend',
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: 'frontend', classifier: '', file: "frontend-${appVersion}.zip", type: 'zip']
                        ]
                    )
               }
            }
        }
        // stage('Deploy') {
        //     steps {
        //         script {
        //             def params = [
        //                     string(name: 'appVersion', value: "${appVersion}")
        //                 ]   
        //                 build job: 'deploy-frontend', parameters: params, wait: false
        //             }
        //     }
        // }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()  #it will delete the workspace after the build run
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
