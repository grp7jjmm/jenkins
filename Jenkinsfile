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
                //snykSecurity additionalArguments: '-d', snykInstallation: 'Snyk', snykTokenId: '813bd878-dd5a-414c-b3e4-d7e300a5f2f1'
                //dependencyCheck additionalArguments: '--scan pom.xml --out /dcheck_reports', odcInstallation: 'Dependency-Check'

            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
                sh 'mvn install checkstyle:checkstyle findbugs:findbugs pmd:pmd'
            }
        }
        
        stage('Test') {
            steps([$class: 'CxScanBuilder', comment: '', configAsCode: true, credentialsId: '4550c17f-8422-4d35-bccb-02e907259154', customFields: '', excludeFolders: '', exclusionsSetting: 'global', failBuildOnNewResults: false, failBuildOnNewSeverity: 'HIGH', filterPattern: '''!**/_cvs/**/*, !**/.svn/**/*, !**/.hg/**/*, !**/.git/**/*, !**/.bzr/**/*,
        !**/.gitgnore/**/*, !**/.gradle/**/*, !**/.checkstyle/**/*, !**/.classpath/**/*, !**/bin/**/*,
        !**/obj/**/*, !**/backup/**/*, !**/.idea/**/*, !**/*.DS_Store, !**/*.ipr, !**/*.iws,
        !**/*.bak, !**/*.tmp, !**/*.aac, !**/*.aif, !**/*.iff, !**/*.m3u, !**/*.mid, !**/*.mp3,
        !**/*.mpa, !**/*.ra, !**/*.wav, !**/*.wma, !**/*.3g2, !**/*.3gp, !**/*.asf, !**/*.asx,
        !**/*.avi, !**/*.flv, !**/*.mov, !**/*.mp4, !**/*.mpg, !**/*.rm, !**/*.swf, !**/*.vob,
        !**/*.wmv, !**/*.bmp, !**/*.gif, !**/*.jpg, !**/*.png, !**/*.psd, !**/*.tif, !**/*.swf,
        !**/*.jar, !**/*.zip, !**/*.rar, !**/*.exe, !**/*.dll, !**/*.pdb, !**/*.7z, !**/*.gz,
        !**/*.tar.gz, !**/*.tar, !**/*.gz, !**/*.ahtm, !**/*.ahtml, !**/*.fhtml, !**/*.hdm,
        !**/*.hdml, !**/*.hsql, !**/*.ht, !**/*.hta, !**/*.htc, !**/*.htd, !**/*.war, !**/*.ear,
        !**/*.htmls, !**/*.ihtml, !**/*.mht, !**/*.mhtm, !**/*.mhtml, !**/*.ssi, !**/*.stm,
        !**/*.bin,!**/*.lock,!**/*.svg,!**/*.obj,
        !**/*.stml, !**/*.ttml, !**/*.txn, !**/*.xhtm, !**/*.xhtml, !**/*.class, !**/*.iml, !Checkmarx/Reports/*.*,
        !OSADependencies.json, !**/node_modules/**/*''', fullScanCycle: 10, password: '{AQAAABAAAAAQuNGYCRPArlBR+H0h74jsRajPIVGbRuv2nSuMOYljV9s=}', projectName: 'WarningPipe', sastEnabled: true, serverUrl: 'http://localhost:8080', sourceEncoding: 'Provide Checkmarx server credentials to see source encodings list', useOwnServerCredentials: true, username: '', vulnerabilityThresholdResult: 'FAILURE', waitForResultsEnabled: true])
            
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
                //h 'contrast auth'
                //sh 'contrast scan'
            }
        }
        
        stage('Release') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '9d2180bc-6df6-4e09-ae05-2a5ca9e590ca', path: '', url: 'http://127.0.0.1:8081/')], contextPath: 'mvnwebapp', onFailure: false, war: '**/*.war'
                script{
                    currentBuild.keepLog = true
                }
                dir("/apache-jmeter-5.5/bin"){
                    sh "./jenkinsreport.sh"
                }
            }
        }            
    }
}
