pipeline {
    agent {
        node {
            label 'docker-agent'
        }
    }
    stages {
        stage('Aqua scanner') {
          agent {
            docker {
              image 'aquasec/aqua-scanner'
            }
          }
          steps {
            git branch: 'main', url: 'https://github.com/mulan04/28571.git'
//            git branch: 'main', credentialsId: '62cdef54-1992-410d-9116-170f28646c0a', url: 'https://github.com/mulan04/28571.git'
// //              
//            git branch: 'main', credentialsId: '62cdef54-1992-410d-9116-170f28646c0a', url: 'https://github.com/mulan04/insecure-banker-app.git'
            withCredentials([
              string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'),
              string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET'),
              string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')
            ]){
              sh '''
                ls -la
                export TRIVY_RUN_AS_PLUGIN=aqua
                trivy fs --scanners config,vuln,secret .
                # To customize which severities to scan for, add the following flag: --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
                # To enable SAST scanning, add: --sast
                # To enable npm/dotnet non-lock file scanning, add: --package-json / --dotnet-proj
              '''
            }
          }
        }
    }
}
