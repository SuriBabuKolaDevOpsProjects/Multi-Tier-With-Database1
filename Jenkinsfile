pipeline {
    agent any
    tools {
        maven 'maven3' 
    }
    environment { 
        SonarQube_Home= tool 'sonarqube'
    }

    stages {
        stage('Git_CheckOut') {
            steps {
                git url: 'https://github.com/SuriBabuKolaDevOpsProjects/Multi-Tier-With-Database.git',
                    branch: 'project1'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Trivy_FS_Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        stage('SonarQube_Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "$SonarQube_Home/bin/sonar-scanner -Dsonar.projectName=MultiTier -Dsonar.projectKey=MultiTier -Dsonar.java.binaries=target"
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Publis_Artifacts_to_Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Nexus_Config', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Build_Docker_Image') {
            steps {
                script  {
                    withDockerRegistry(credentialsId: 'Docker_Cred', toolName: 'docker') {
                        sh "docker image build -t nagasuribabukola/bankapp:${BUILD_NUMBER} -t nagasuribabukola/bankapp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy_Image_Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html nagasuribabukola/bankapp:${BUILD_NUMBER}"
            }
        }
        
        stage('Push_Docker_Image') {
            steps {
                script  {
                    withDockerRegistry(credentialsId: 'Docker_Cred', toolName: 'docker') {
                        sh "docker image push nagasuribabukola/bankapp:${BUILD_NUMBER}"
                        sh "docker image push nagasuribabukola/bankapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script  {
                    // Stop existing containers to ensure a fresh start
                    sh 'docker-compose down'
                    // Start containers in detached mode
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}
