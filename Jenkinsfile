pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
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
        // stage('Checkout from Git'){
        //     steps{
        //         git 'https://github.com/gaur8av/Movies-Music-Site.git'
        //     }
        // }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/deepeshmlgupta/Movies-Music-Site.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.mandmjectName=mandm \
                    -Dsonar.mandmjectKey=mandm '''
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                     sh "docker build -t mandm:latest ."
                     sh "docker tag mandm deepesh/mandm:latest "
                //   withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker'){   
                //       sh "docker build -t mandm:latest ."
                //       sh "docker tag mandm deepesh/mandm:latest "
                //     //   sh "docker push deepesh/swiggy:latest "
                //     }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image deepesh/mandm:latest > trivy.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name mandm -p 3000:3000 mandm:latest'
            }
        }
    }
}