pipeline {
    agent { label 'paresh'}
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('git clone') {
            steps {
                git branch: 'aws-cicd', url: 'https://github.com/Routparesh/node-todo-cicd.git'
            }
        }

        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=nodeapp \
                    -Dsonar.projectKey=nodeapp '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage("TRIVY File scan"){
            steps{
                sh "trivy fs . > trivy-fs_report.txt"
            }
        }

        stage("OWASP Dependency Check"){
            withCredentials([usernamePassword(credentialsId: 'nvd-api-key', passwordVariable: 'nvd-Cred', usernameVariable: 'nvd-Var')]) {
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
       }
        
    }
}
