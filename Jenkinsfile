pipeline {
    agent any
    stages {
        stage ('Build landingpage') {
            steps {
                sh '''
                    sudo docker build -t tomyhp/landingpage:$GIT_BRANCH-$BUILD_ID -f landing/Dockerfile .
                    sudo docker login -u tomyhp -p$DOCKER_TOKEN
                    sudo docker push tomyhp/landingpage:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('Change manifest file and send') {
            steps {
                sh '''
                    sed -i -e "s/branch/$GIT_BRANCH/" Kube/landing-page/landing-page-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" Kube/landing-page/landing-page-deployment.yml
                    tar -czvf manifest.tar.gz Kube/*
                '''
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "kube-master-tomy",
                            transfers: [sshTransfer(sourceFiles: 'manifest.tar.gz', remoteDirectory: 'jenkins/')],
                            verbose: true
                        )
                    ]
                )
            }
        }
        stage ('Deploy to kubernetes cluster') {
            steps {
                sshagent(credentials : ['kube-master-tomy']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id tar -xvzf jenkins/manifest.tar.gz'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f ./Kube/namespace/'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f ./Kube/landing-page'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f ./Kube/ingress-nginx/'
                }
            }
        }
    }
}
