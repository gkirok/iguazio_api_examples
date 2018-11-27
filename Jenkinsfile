def label = "${UUID.randomUUID().toString()}"
def BUILD_FOLDER = '/go'
def github_user = "gkirok"
def docker_user = "gallziguazio"


properties([pipelineTriggers([[$class: 'PeriodicFolderTrigger', interval: '2m']])])
podTemplate(label: "netops-demo-${label}", inheritFrom: 'kube-slave-dood') {
    node("netops-demo-${label}") {
        withCredentials([
                usernamePassword(credentialsId: '4318b7db-a1af-4775-b871-5a35d3e75c21', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')
        ]) {
            stage('release') {
                def TAG_VERSION = sh(
                        script: "echo ${TAG_NAME} | tr -d '\\n' | egrep '^v[\\.0-9]*.*\$' | sed 's/v//'",
                        returnStdout: true
                ).trim()
                if ( TAG_VERSION ) {
//                    def V3IO_TSDB_VERSION = sh(
//                            script: "echo ${TAG_VERSION} | awk -F '-v' '{print \"v\"\$2}'",
//                            returnStdout: true
//                    ).trim()

                    stage('prepare sources') {
                        sh """ 
                                cd ${BUILD_FOLDER}
                                git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${github_user}/iguazio_api_examples.git src/github.com/v3io/iguazio_api_examples
                                cd ${BUILD_FOLDER}/src/github.com/v3io/iguazio_api_examples/netops_demo/golang/src/github.com/v3io/demos
                                rm -rf vendor/github.com/v3io/v3io-tsdb/
                                git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${github_user}/v3io-tsdb.git vendor/github.com/v3io/v3io-tsdb
                                cd vendor/github.com/v3io/v3io-tsdb
                                rm -rf .git vendor/github.com/v3io vendor/github.com/nuclio
                        """
                    }
//                                git checkout ${V3IO_TSDB_VERSION}

                    stage('build in dood') {
                        container('docker-cmd') {
                            sh """
                                cd ${BUILD_FOLDER}/src/github.com/v3io/iguazio_api_examples/netops_demo/golang/src/github.com/v3io/demos
                                docker build . --tag netops-demo-golang:latest --tag ${docker_user}/netops-demo-golang:$NETOPS_DEMO_VERSION --build-arg NUCLIO_BUILD_OFFLINE=true --build-arg NUCLIO_BUILD_IMAGE_HANDLER_DIR=github.com/v3io/demos

                                cd ${BUILD_FOLDER}/src/github.com/v3io/iguazio_api_examples/netops_demo/py
                                docker build . --tag netops-demo-py:latest --tag ${docker_user}/netops-demo-py:$NETOPS_DEMO_VERSION
                            """
                            withDockerRegistry([credentialsId: "472293cc-61bc-4e9f-aecb-1d8a73827fae", url: ""]) {
                                sh "docker push ${docker_user}/netops-demo-golang:${NETOPS_DEMO_VERSION}"
                                sh "docker push ${docker_user}/netops-demo-py:${NETOPS_DEMO_VERSION}"
                            }
                        }
                    }

                    stage('git push') {
                        try {
                            sh """
                                git config --global user.email '${GIT_USERNAME}@iguazio.com'
                                git config --global user.name '${GIT_USERNAME}'
                                cd ${BUILD_FOLDER}/src/github.com/v3io/iguazio_api_examples/netops_demo
                                git add *
                                git commit -am 'Updated TSDB to latest';
                                git push origin master
                            """
                        } catch (err) {
                            echo "Can not push code to git"
                        }
                    }
                } else {
                    echo "${TAG_VERSION} is not release tag."
                }
            }
        }
    }
}