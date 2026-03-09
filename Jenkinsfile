pipeline {
    agent { label 'paresh' }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME   = 'routparesh/node-app'
        IMAGE_TAG    = 'latest'
    }

    stages {

        stage('Git Clone') {
            steps {
                git branch: 'aws-cicd',
                    url: 'https://github.com/Routparesh/node-todo-cicd.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                          $SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true,
                        credentialsId: 'sonar-token'
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                  trivy fs . \
                  --severity HIGH,CRITICAL \
                  --exit-code 0 \
                  > trivy-fs.txt
                '''
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck(
                        additionalArguments: '''
                          --scan . \
                          --format XML \
                          --nvdApiKey $NVD_API_KEY
                        ''',
                        odcInstallation: 'DP-Check'
                    )
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      docker login -u $DOCKER_USER -p $DOCKER_PASS
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                  trivy image $IMAGE_NAME:$IMAGE_TAG \
                  --severity HIGH,CRITICAL \
                  --exit-code 0 \
                  > trivy-image.txt
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                  docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
