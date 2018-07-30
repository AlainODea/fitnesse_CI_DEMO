import groovy.json.*
pipeline {
    agent any
    environment {
          def scannerHome = tool 'sonar_scanner'
          def prNo = "${CHANGE_ID}"
          def repo_url = "${env.GIT_URL}"
          def repo_name = repo_url.replace("https://github.com/","").replace(".git","")
          def jenkins_credentials_ID = 'Carter-Admin'
    }
    stages {
        stage('Build') {
            steps {
                  sh './gradlew'
            }
        }
        /*
        stage('Test') {
            steps {
                  sh './gradlew test'
            }
        }
        
        stage('Sonarqube'){
            when {
                expression {
                    "${prNo}" != "null"
                }
            }
            steps{
                withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.analysis.mode=preview "+
                            "-Dsonar.github.repository=${repo_name} "+
                            "-Dsonar.github.pullRequest=${prNo} "+
                            "-Dsonar.github.oauth=${GITHUB_TOKEN} "+
                            "-Dsonar.github.endpoint=https://api.github.com/"
                    }
                }
            }
        }
        */
        stage('Check Security Risk'){
            when {
                expression {
                    "${prNo}" != "null"
                }
            }
            steps{
                script{
                    try{
                        def changed_files = getChangedFiles()
                        echo changed_files.filename.toString()
                        def files = changed_files.filename
                        if(fileMatches{ fileNames = "$files" }){
                           echo 'File matches, reviewing PR...'
                           postReview(getLowsecriskComment())
                        }
                    }catch(Exception e){
                     echo e.getMessage()   
                    }
                }
            }
        }
    }
}

def parseJson(jsonText) {
    json_map = readJSON text: jsonText
    return json_map
}

def getChangedFiles(){
    script{
        withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
            /*def pr_files = sh (script: 'curl -H "Content-Type: application/json" -H "Authorization: bearer GITHIB_TOKEN" -X -d \
                \'{"query": "query { \
                    repository(Owner: \"JenkinsSonarQubeTesting\", name: \"fitnesse_CI_DEMO\"){ \
                        pullRequests(states:[OPEN, MERGED], last:10){ \
                            edges { \
                                node { \
                                    reviews(first: 10, states:[COMMENTED, APPROVED]) { \
                                        nodes { \
                                            createdAt \
                                            bodyText \
                                            author { \
                                                login \
                                            } \
                                        } \
                                    } \
                                    url \
                                } \
                            } \
                            pageInfo { \
                                startCursor hasPreviousPage \
                            } \
                        } \
                    } \
                }" https://api.github.com/graphql\'', returnStdout: true).trim()
                */                                                                       
                                                                                                
            def pr_files = sh (script: "curl -s -H \"Authorization: token ${GITHUB_TOKEN}\" \"https://api.github.com/repos/${repo_name}/pulls/${prNo}/files\"", returnStdout: true).trim()
            return parseJson(pr_files)
        }
    }
}

def postReview(message){
    script{
        withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
            sh "curl -s -H \"Authorization: token ${GITHUB_TOKEN}\" -X POST -d '{\"body\": \"${message}\", \"event\": \"COMMENT\"}' \"https://api.github.com/repos/${repo_name}/pulls/${prNo}/reviews\""
        }
    }
}

def getLowsecriskComment(){
    return 'Files match (Automated review)'
}
