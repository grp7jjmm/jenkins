pipeline {
    agent any
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
    }
    stages {
        stage('GitHub') {
            options {
                timeout(time: 1, unit: 'MINUTES')
            }
            steps {
                script {
                    env.URL = input message: 'Place your Github Repository here:', parameters: [string(defaultValue: '', name: 'Repository URL')]
                }
                git "${env.URL}"
                snykSecurity additionalArguments: '-d', snykInstallation: 'Snyk', snykTokenId: '813bd878-dd5a-414c-b3e4-d7e300a5f2f1'
                dependencyCheck additionalArguments: '--scan pom.xml --out /dcheck_reports', odcInstallation: 'Dependency-Check'

            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
                sh 'mvn install checkstyle:checkstyle findbugs:findbugs pmd:pmd'
            }
        }
        
        stage('Test') {
            steps{
                withSonarQubeEnv('SonarQube') { 
        // If you have configured more than one global server connection, you can specify its name
        //      sh "${scannerHome}/bin/sonar-scanner"
                    sh "mvn sonar:sonar"
                }
                recordIssues(
                    enabledForFailure: true, aggregatingResults: true, 
                    tools: [java(), checkStyle(), findBugs(), pmdParser()]
                )
                sh 'contrast auth'
                sh 'contrast scan'
            }
        }
        
        stage('Release') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '9d2180bc-6df6-4e09-ae05-2a5ca9e590ca', path: '', url: 'http://127.0.0.1:8081/')], contextPath: 'mvnwebapp', onFailure: false, war: '**/*.war'
                script{
                    currentBuild.keepLog = true
                }
            }
        }            
    }
}
