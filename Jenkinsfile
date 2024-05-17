
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
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/root/.cache/ aquasec/trivy:latest --quiet image yasreebakmal/myntra:latest  -f json -o /root/.cache/results.json'
                }
            }
         }
        }
        stage('Send result to AccuKnox') {
            steps {
                dir ('/var/lib/jenkins/workspace/Hotstar') {
                    sh '''curl --location --request POST 'https://cspm.demo.accuknox.com/api/v1/artifact/?tenant_id=2410&data_type=TR&save_to_s3=false' \
                    --header 'Tenant-Id: 2410' \
                    --header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzE4NTExODAxLCJqdGkiOiI5NTc5ZTZlZThiYTQ0ZjgyYjYwMjc4ODJhOWQ0NzNjMyIsImlzcyI6ImNzcG0uZGVtby5hY2N1a25veC5jb20ifQ.jPBbJqrw1806w1xtdxEo4Ihlv-Wsp6PLj2iM0HK63489LxpAt2akvetVb2cfysoYtNv2ISjYbqHh1JDAYF3ZZMgYewHpgLv-bjVCasYqAlFN-jD4AOs78youtvfc2CIKrwXXl5l7-_O5IkNJRZ8bXb-MRlyoMJfnOPwPgzu3rhJLPjxt3Wx_be0nZgzOiE_3IH2RfXyv1PVbBZB3i7g1wjQGYQKzIGBYxX8TB_-QPt4CC03zUT1QHYayMhQD2C4PiUOPFSXyk0vx8p7iMwAnJx-53_Z1UmExwpKiLHAqBNgIoLaPDqW-anaZs4Xr6W3i6WJgYbr8gw_btZGm8vLFrw' \
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
