
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node19'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/yasreebsaad/Hotstar-Clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=myntra \
                    -Dsonar.projectKey=myntra'''
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
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                 //dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                sh 'pwd'
            }
        }
        stage('Docker Scout FS') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       //sh 'docker-scout quickview fs://.'
                       //sh 'docker-scout cves fs://.'
                       sh 'docker --version'
                   }
                }   
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t myntra ."
                       sh "docker tag myntra yasreebakmal/myntra:latest "
                       sh "docker push yasreebakmal/myntra:latest"
                    }
                }
            }
        }
        stage('Install and run ImageScan') {
            steps {
              dir ('/var/lib/jenkins/workspace/Hotstar') {
                script {
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest --quiet image yasreebakmal/myntra:latest -f json -o results.json'
                }
            }
         }
        }
        stage('Send result to AccuKnox') {
            steps {
                dir ('/var/lib/jenkins/workspace/Hotstar') {
                    sh '''curl --location --request POST 'https://cspm.demo.accuknox.com/api/v1/artifact/?tenant_id=<tenantID>&data_type=TR&save_to_s3=false' \
                    --header 'Tenant-Id: 2410' \
                    --header 'Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzQ3Mjk4MDMwLCJqdGkiOiIwNjU5ZmQ2ZWEwNDQ0NjVlOTY4NzAwZTQ5Zjk2YzhmYyIsImlzcyI6ImNzcG0uZGVtby5hY2N1a25veC5jb20ifQ.eGJ5-uSM6KK0OG7PIFoOfTEkleNfwDV0K-Nsqz-0His3qOm9RFjqGnd6Cuo6XmNljz691WNu_E16uioS_TiyCJ3M0fFev06joLa60P98feIGm0Egs5RO-eN6x9cdApbKDChftWheJU_D0iXr6QVIg0y7ZWK2O9AfBKmWmOUmYO7jEFFFbCkjvjIg2RD7MD24KNKkuvpgjC75TIDErRz3yRnbPzt5XWAxdw7DmKSWMNZ-2kjEwOydw5x_5TJTsKlJoxTtgY1dAWxwGTPSs92cC_xiNDKqc3xBhEiydfSjCdsPmm5WoYbIVZYU4pkcSx9UzP5VBpsNCWTpX3PntfNaaQ' \
                    --form 'file=@"./results.json"'
                    '''        
                }
            }
          }

        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview yasreebakmal/myntra:latest'
                       sh 'docker-scout cves yasreebakmal/myntra:latest'
                       sh 'docker-scout recommendations yasreebakmal/myntra:latest'
                   }
                }   
            }
        }
          stage('TRIVY FS SCAN') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                      // sh 'docker-scout quickview yasreebakmal/myntra:latest'
                       //sh 'docker-scout cves yasreebakmal/myntra:latest'
                       //sh 'docker-scout recommendations yasreebakmal/myntra:latest'
                      
                       sh 'trivy image yasreebakmal/myntra:latest > trivyfs.json'
                   }
                }
                
            }
        }
        stage("Remove container"){
            steps{
                sh "docker stop myntra | true"
                sh "docker rm myntra | true"
            }
        }
        stage("deploy_docker"){
            steps{
                sh "docker run -d --name myntra -p 3000:3000 yasreebakmal/myntra:latest"
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('K8S') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }   
                    }
                }
            }
        }
    }
}
