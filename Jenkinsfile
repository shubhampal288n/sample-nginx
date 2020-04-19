node {
    def app
    def label = "docker-jenkins-test"
    def organisation = "lolc";
    def appName = "sample-app"
    def tag = "$organisation/$appName"
    def registryUrl = "298647176693.dkr.ecr.ap-southeast-1.amazonaws.com"
    def region = "ap-southeast-1"
    def repo = "github.com/dunzoit/eagle.git"
    def namespace = "default"

    podTemplate(
        label: label,
        showRawYaml: false,
        containers: [
            containerTemplate(name: 'docker', image: 'docker:latest', command: 'cat', ttyEnabled: true),
            containerTemplate(name: 'kubectl', image: 'bskim45/helm-kubectl-jq:latest', command: 'cat', ttyEnabled: true),
        ],
        volumes: [
            hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        ])
    {
        node(label) {
                stage ('Checkout Code') {
                    git branch: 'master', credentialsId: 'fb861711-0994-4ab0-bbf6-b5394683b207', url: 'ssh://https://git.hashedin.com/shubham.pal/nginx-sample.git'
                    }
                stage('Docker Build & Push Image') {
                    container('docker') {
                        docker.withRegistry("https://$registryUrl", "ecr:$region:ECR_ACCESS") {
                           // sh "docker build --build-arg GITHUB_TOKEN=$GITHUB_TOKEN --network=host -t $registryUrl/$tag:$GIT_COMMIT_HASH ."
                            sh """
                            echo "--> Pull service latest image if exists"    
                            docker pull $registryUrl/$tag:latest || true
                            echo "--> Build service latest image using latest service and builder cache"   
                            docker build \
                                --network=host \
                                --cache-from $registryUrl/$tag:latest \
                                -t $registryUrl/$tag:$GIT_COMMIT_HASH \
                                -t $registryUrl/$tag:latest \
                                .
                            echo "--> Push service latest & versioned image to registry" 
                            docker push $registryUrl/$tag:latest
                            docker push $registryUrl/$tag:$GIT_COMMIT_HASH
                            """
                        }
                    }
                }

            }
        }
    }
