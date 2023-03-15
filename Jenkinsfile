pipeline {
    agent {
        node {
            label 'docker-agent'
        }
    }
    stages {
        stage('Checkout') { 
            steps {
                git branch: 'main', credentialsId: '62cdef54-1992-410d-9116-170f28646c0a', url: 'https://github.com/aquasupportemea/insecure-bank-app'
                // geht nicht - git 'https://github.com/mulan04/aqua-jenkins-demo'
                // sh 'curl https://raw.githubusercontent.com/mulan04/aqua-jenkins-demo/main/Dockerfile -o Dockerfile'
            }
        }
        stage('Build') { 
            steps {
                sh '''
                    echo Build
                    docker info
                    docker build -t mulan04/${JOB_NAME}:${BUILD_NUMBER} .
                    docker tag mulan04/${JOB_NAME}:${BUILD_NUMBER} mulan04/${JOB_NAME}:latest
                '''
            }
        }
        stage('Scan') { 
            steps {
                aqua    containerRuntime: 'docker', 
                        customFlags: '--show-negligible --checkonly --no-verify --html', 
                        hideBase: false, hostedImage: '', 
                        localImage: 'mulan04/${JOB_NAME}:${BUILD_NUMBER}', 
                        localToken: 'f175b78c210c6cca8ee8a49baa4b55b4b59ac2cb', 
                        locationType: 'local', 
                        notCompliesCmd: '', onDisallowed: 'ignore', policies: '', register: false, registry: '', 
                        scannerPath: '', showNegligible: false, tarFilePath: ''
/*
                sh '''
                    echo Scan
                    docker run \
                        -e BUILD_JOB_NAME=MyJenkinsDemo \
                        -e BUILD_URL=http://mulan04.daheim.com:8090/job/Declarative%20Pipeline/${BUILD_NUMBER}/ \
                        -e BUILD_NUMBER=${BUILD_NUMBER} \
                        --privileged --dns=192.168.0.148 --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        registry.aquasec.com/scanner:2022.4 scan \
                        --show-negligible --checkonly --no-verify --html \
                        --host http://mulan04.daheim.com:8080 \
                        --token f175b78c210c6cca8ee8a49baa4b55b4b59ac2cb \
                        --local mulan04/test:${BUILD_NUMBER}
                '''
*/
            }
        }
        stage('Manifest Generation') {
            steps {
                // Replace GITHUB_APP_CREDENTIALS_ID with the id of your github app credentials
                withCredentials([
                    usernamePassword(credentialsId: 'GITHUB_APP_CREDENTIALS_ID', usernameVariable: 'GITHUB_APP', passwordVariable: 'GITHUB_TOKEN'), 
                    string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'), 
                    string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET')
                ]) {
                    // Replace ARTIFACT_PATH with the path to the root folder of your project 
                    // or with the name:tag the newly built image
                    sh '''
                        export BILLY_SERVER=https://billy.codesec.aquasec.com
                        curl -sLo install.sh download.codesec.aquasec.com/billy/install.sh
                        curl -sLo install.sh.checksum https://github.com/argonsecurity/releases/releases/latest/download/install.sh.checksum
                        if ! cat install.sh.checksum | sha256sum ; then
                            echo "install.sh checksum failed"
                            exit 1
                        fi
                        BINDIR="." sh install.sh
                        rm install.sh install.sh.checksum
                        ./billy generate \
                            --access-token ${GITHUB_TOKEN} \
                            --aqua-key ${AQUA_KEY} \
                            --aqua-secret ${AQUA_SECRET} \
                            --artifact-path "mulan04/${JOB_NAME}:${BUILD_NUMBER}"
                            
                            # The docker image name:tag of the newly built image
                            # --artifact-path "my-image-name:my-image-tag"
                            # OR the path to the root folder of your project. I.e my-repo/my-app 
                            # --artifact-path "ARTIFACT_PATH"
                    '''
                }
            }
        }       
        stage('Aqua scanner') {
          agent {
            docker {
              image 'aquasec/aqua-scanner'
            }
          }
          steps {
            git branch: 'main', credentialsId: '62cdef54-1992-410d-9116-170f28646c0a', url: 'https://github.com/aquasupportemea/insecure-bank-app'
            withCredentials([
              string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'),
              string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET'),
              string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')
            ]){
              sh '''
                export TRIVY_RUN_AS_PLUGIN=aqua
                trivy fs --scanners config,vuln,secret .
                # To customize which severities to scan for, add the following flag: --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
                # To enable SAST scanning, add: --sast
                # To enable npm/dotnet non-lock file scanning, add: --package-json / --dotnet-proj
              '''
            }
          }
        }
        stage('Deploy') { 
            steps {
                sh '''
                    echo Deploy
                    docker images
                    env
                '''
            }
        }
    }
}
