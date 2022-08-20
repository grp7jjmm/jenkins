pipeline {
    agent any
    
    // environment for sonarqube
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
    }
    
    // only one build can be run at a time
    options {
      disableConcurrentBuilds abortPrevious: true
    }
    
    stages {
        
        // This stage pulls the client's Github Repository
        stage('GitHub') {
            
            // This build will timeout after 2 mins of idle interaction
            options {
                timeout(time: 2, unit: 'MINUTES')
            }
            
            // An input field will be produced to allow clients to put their Github repository URL
            steps {
                script {
                    env.URL = input message: 'Place your Github Repository here:', parameters: [string(defaultValue: '', name: 'Repository URL')]
                }
                git "${env.URL}"
            }
        }
        
        // This stage will package the maven project into a .war file
        stage('Build') {
            steps {
                
                // Snyk and DependencyCheck will scan the source code of the project for vulnerabilities and unneccesary dependencies
                snykSecurity additionalArguments: '-d', failOnError: false, failOnIssues: false, snykInstallation: 'Snyk', snykTokenId: '813bd878-dd5a-414c-b3e4-d7e300a5f2f1'
                dir ("/jenkins-reports/.scripts"){
                    sh "./snyk.sh"
                }
                dependencyCheck additionalArguments: '--scan pom.xml --out /dcheck_reports --format HTML', odcInstallation: 'Dependency-Check'
                dir ("/jenkins-reports/.scripts"){
                    sh "./dcheck.sh"
                }
                
                // This script runs the .jar file dependency check
                dir ("/jenkins-reports/.scripts"){
                    sh "./jarcheck.sh"
                }
                
                // After the scan, the project will be compiled into a war file
                dir ("/jenkins-reports/.scripts"){
                    sh "./mvn_war.sh"
                }
                
                // Compiling the client's project into a .war file
                sh 'mvn compile war:war'
                
                // Install WarningNextGen plugins for Testing stage
                sh 'mvn install checkstyle:checkstyle findbugs:findbugs pmd:pmd'
                
                // Fingerprinting and hashing of the war file 
                fingerprint '**/*.war'
                dir ("/jenkins-reports/.scripts"){
                    sh "./sha256hash.sh"
                }
            }
        }
        
        stage('Test') {
            steps{
                
                // Initializing SonarQube static analysis tool 
                withSonarQubeEnv('SonarQube') { 
                    sh "mvn sonar:sonar"
                }
                
                // Initializing the plugins of the WarningNextGen security tool for hybrid analysis
                recordIssues(
                    enabledForFailure: true, aggregatingResults: true, 
                    tools: [java(), checkStyle(), findBugs(), pmdParser()]
                )
            }
        }
        
        // This stage will release the compiled .war file to Tomcat
        stage('Release') {
            steps {
                
                // Checking if the .war file created is still the legitimate one from the Build Stage.
                fingerprint '**/*.war'
                
                // The build will fail if the hash values do not match
                script{
                    check = sh(script: "/jenkins-reports/.scripts/checkhash.sh", returnStatus: true)
                    if (check != 0) {
                        currentBuild.result = 'FAILURE'
                    }
                }
                
                // Deploying the .war file to Tomcat (port 8081) 
                deploy adapters: [tomcat8(credentialsId: '9d2180bc-6df6-4e09-ae05-2a5ca9e590ca', path: '', url: 'http://localhost:8081/')], contextPath: 'mvnwebapp', onFailure: false, war: '**/*.war'
                
                // Initialing the debugger within JMeter for dynamic analysis
                dir("/apache-jmeter-5.5/bin"){
                    sh "./jenkinsreport.sh"
                }
                
                dir ("/jenkins-reports/.scripts"){
                    sh "./owaspzap.sh"
                }
                
                // Link to the main HTML report
                echo "If you would like to see the reports of the various security tools, you can find them here -> file:///jenkins-reports/index.html"
            }
        }            
    }
}
