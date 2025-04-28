pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'JDK21'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git CheckOut') {
            steps {
                git branch: 'main', url: 'https://github.com/vijaykanth1729/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Testing') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                -Dsonar.java.binaries=. '''
            }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '***/dependency-check-report.xml'
            }
        }
        stage('Build Application') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
         stage('Deploy Artifact to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'JDK21', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker Build & Tag Image') {
            steps {
               script {
                   // This step should not normally be used in your script. Consult the inline help for details.
                withDockerRegistry(credentialsId: 'docker-creds', url:'') {
                    sh "docker build -t iamvijay/ekart:latest -f docker/Dockerfile ."
                    }
               }
               }
            }
    stage('Perform Trivy Scan') {
            steps {
                sh "trivy image iamvijay/ekart:latest > trivy-report.txt"
            }
        }
    stage('Docker Push Image') {
            steps {
               script {
                   // This step should not normally be used in your script. Consult the inline help for details.
                withDockerRegistry(credentialsId: 'docker-creds', url:'') {
                    sh "docker push iamvijay/ekart:latest"
                    }
               }
               }
            }
   stage('Hello WORLD') {
            steps {
                echo "Hello World"
            }
        }         
}
}
