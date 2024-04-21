pipeline {
    agent any 
    tools {
        maven "maven3"
        jdk "jdk17"
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Pawan-choudhary/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage("Unit Tests") {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projecKey=EKART -Dsonar.projecName=EKART \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Build") {
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
        stage("Deploy to Nexus") {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven',jdk:'jdk17', maven:'maven3', mavenSettingsConfig:'',traceability:true){
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage("Docker Build & Tag Image") {
            steps {
                script {
                    withDockerRegistry(credentialsID: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t pawanchoudharynain/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                sh "trivy image pawanchoudharynain/ekart:latest > trivy-report.txt" // scan the image and put output in trivy-report.txt
            }
        }
        stage("Pushing Image  ") {
            steps {
                script {
                    withDockerRegistry(credentialsID: 'docker-cred', toolName: 'docker') {
                        sh "docker push pawanchoudharynain/ekart:latest"
                    }
                }
            }
        }
        stage("Kubernetes Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess:false, serverUrl: 'https://IP:Port') {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                }

            }
        }
    }
}