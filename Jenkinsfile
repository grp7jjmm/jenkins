pipeline {
    agent any
    
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
    }
    options {
      disableConcurrentBuilds abortPrevious: true
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
            }
        }
        
        stage('Build') {
            steps {
                snykSecurity additionalArguments: '-d', failOnError: false, failOnIssues: false, snykInstallation: 'Snyk', snykTokenId: '813bd878-dd5a-414c-b3e4-d7e300a5f2f1'
                dependencyCheck additionalArguments: '--scan pom.xml --out /jenkins-reports/dependency-check', odcInstallation: 'Dependency-Check'
                dir ("/var/lib/jenkins/workspace/Input/target"){
                    sh "./mvn_war.sh"
                }
                sh 'mvn compile war:war'
                fingerprint '**/*.war'
                sh 'mvn install checkstyle:checkstyle findbugs:findbugs pmd:pmd'
            }
        }
        
        stage('Test') {
            steps{
                withSonarQubeEnv('SonarQube') { 
                    sh "mvn sonar:sonar"
                }
                recordIssues(
                    enabledForFailure: true, aggregatingResults: true, 
                    tools: [java(), checkStyle(), findBugs(), pmdParser()]
                )
            }
        }
        
        stage('Release') {
            steps {
                fingerprint '**/*.war'
                deploy adapters: [tomcat8(credentialsId: '9d2180bc-6df6-4e09-ae05-2a5ca9e590ca', path: '', url: 'http://localhost:8081/')], contextPath: 'mvnwebapp', onFailure: false, war: '**/*.war'
                dir("/apache-jmeter-5.5/bin"){
                    sh "./jenkinsreport.sh"
                }
            }
        }            
    }
}
