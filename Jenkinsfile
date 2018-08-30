node {
 
    withEnv(["PATH+MAVEN=${tool 'apache-maven-3.5.3'}bin"]) {
 
        stage ('Checkout') {
            checkout scm
        
        }
 
         stage('Build') {
               def pom = readMavenPom file: 'pom.xml'
            print "Build: " + pom.version
            env.POM_VERSION = pom.version
            sh 'mvn clean test -Dmaven.test.failure.ignore=true'
            currentBuild.description = "v${pom.version} (${env.branch})"
				
        }
       stage('QA') {
            withSonarQubeEnv('sonar') {
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
            }
        }
        stage('JFROG') {
            def server = Artifactory.server "jfrog-artifactory"
            def buildInfo = Artifactory.newBuildInfo()
            def rtMaven = Artifactory.newMavenBuild()
            rtMaven.tool = 'apache-maven-3.5.3'
            rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
            rtMaven.resolver releaseRepo:'remote-repos', snapshotRepo:'remote-repos', server: server
            rtMaven.run pom: 'pom.xml', goals: 'clean install -Dmaven.test.skip=true', buildInfo: buildInfo
            publishBuildInfo server: server, buildInfo: buildInfo
        }
        stage('Input') {
                input('Do you want to proceed?')
        }

        stage('If Proceed is clicked') {
                print('hello')
        } 
        stage('Push to cloudfoundry') {
                pushToCloudFoundry cloudSpace: 'dev', credentialsId: 'b7c24062-6ea4-4876-89a1-96b3c2b430b2', manifestChoice: [manifestFile: '/var/lib/jenkins/workspace/sample_full_cloudfoundry/manifest.yml'], organization: ('techm_dev'), selfSigned: ('true'), target: 'api.app-cloudfoundry.com'
                
        }
        }
    }
