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
        
        
        stage('Build') {
            steps{
                
                // Compiling the client's project into a .war file
                sh 'mvn compile war:war'
               
                // Fingerprinting and hashing of the war file 
                fingerprint '**/*.war'
                dir ("/jenkins-scripts/.scripts"){
                    sh "./sha256hash.sh"
                }
                
            }
        }
        
        // This stage will release the compiled .war file to Tomcat
        stage('Release') {
            steps {
                
                // Checking if the .war file created is still the legitimate one from the Build Stage.
                fingerprint '**/*.war'
                
                // The build will fail if the hash values do not match
                script{
                    check = sh(script: "/jenkins-scripts/.scripts/checkhash.sh", returnStatus: true)
                    if (check != 0) {
                        currentBuild.result = 'FAILURE'
                    }
                }
                
                // Deploying the .war file to Tomcat (port 8081) 
                deploy adapters: [tomcat8(credentialsId: '9d2180bc-6df6-4e09-ae05-2a5ca9e590ca', path: '', url: 'http://localhost:8081/')], contextPath: 'mvnwebapp', onFailure: false, war: '**/*.war'
               
            }
        }
        
        
        
    }
}
