pipeline {
    agent any
    stages {
        stage ('Build landingpage') {
            steps {
                sh '''
                    sudo docker build -t tomyhp/landing-page:$GIT_BRANCH-$BUILD_ID -f landing/Dockerfile .
                    sudo docker login -u tomyhp -p$DOCKER_TOKEN
                    sudo docker push tomyhp/landing-page:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('Change manifest file and send') {
            steps {
                script {
		If ($GIT_BRANCH == "main"){
                    sed -i -e "s/branch/$GIT_BRANCH/" Kube-production/landing-page/landing-page-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" Kube-production/landing-page/landing-page-deployment.yml
                    tar -czvf manifest.tar.gz Kube-production/*
                    } else {
                    sed -i -e "s/branch/$GIT_BRANCH/" Kube-staging/landing-page/landing-page-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" Kube-staging/landing-page/landing-page-deployment.yml
                    tar -czvf manifest.tar.gz Kube-staging/*
                    }
                } 
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
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f ./Kube/Kube-production/'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f ./Kube/Kube-staging/'
                }
            }
        }
    }
}
