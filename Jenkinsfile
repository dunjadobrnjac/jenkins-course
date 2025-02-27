def sendEmail(name, id, status, output){
    echo (message: "Send email from ${name} with status ${status}.")
    echo (message: "Output: ${output}")
}

def output = ""

pipeline {
    agent any
    
    parameters {
        string defaultValue: 'hello_world.zip', name: 'ARTIFACT_NAME', trim: true
        booleanParam defaultValue: false, name: 'FAIL_PIPELINE'
        booleanParam defaultValue: true, name: 'RUN_TEST'
    }
    
    stages {
        stage('Download'){
            steps {
                cleanWs()
                echo (message: "Download")
                dir('pipeline'){
                    git (
                    branch: 'pipeline',
                    url: 'https://github.com/KLevon/jenkins-course.git'
                    )
                }
                
                rtDownload(
                    serverId: 'Artifactory',
                    spec: '''{ 
                        "files": [
                        {
                            "pattern":"generic-local/libraries/printer.zip",
                            "target":"./"
                        }
                        ]
                    }'''
                )
                
                unzip (
                    zipFile: "libraries/printer.zip",
                    dir: "pipeline/"
                )
            }
        }
        stage('Build'){
            steps {
                echo (message: "Build")
                dir('pipeline'){
                    bat (
                        script: """
                            Makefile.bat
                        """
                    )
                }
                
                withCredentials(
                    [usernamePassword(credentialsId:'CRED_DUNJA', passwordVariable:'psw', usernameVariable:'usr')]
                ){
                    echo (message: "Username: ${usr} and password: ${psw}")
                }
            }
        }
        
        stage('Tests'){
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                
                script {
                    def array = ["printer", "scanner", "main"]
                    for(element in array){
                        output += bat (
                            script: """
                                pipeline/Tests.bat $element
                            """,
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }
        
        stage('Publish'){
            steps {
                echo (message: "Publish")
                script {
                    zip (
                        //zipFile: "hello_world.zip",
                        zipFile: """${params.ARTIFACT_NAME}""",
                        archive: true,
                        dir: "pipeline/",
                    )
                }
                
                rtUpload (
                    serverId: 'Artifactory',
                    spec: """{ 
                        "files": [
                        {
                            "pattern": "${params.ARTIFACT_NAME}",
                            "target":"generic-local/release/dunja/${env.BUILD_ID}/"
                        }
                        ]
                    }"""
                )
                
                script{
                    if(params.FAIL_PIPELINE==true){
                        bat (
                            script: """
                                exit 1
                            """
                        )
                    }
                }
            }
        }

        stage('Dynamic'){
            when {
                branch 'feature/multi'
            }
            steps {
                echo (message: "Dynamic")
            }
        }
    }
    
    post {
        failure {
            script {
                sendEmail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
            }
        }
    }
}





