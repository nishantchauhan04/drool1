podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'maven', 
            image: 'maven:3-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        def repository
        stage ('Docker-Build') {
            container ('docker') {
                withDockerRegistry([credentialsId: 'dockerhub']) {
                    sh "docker build -t 172.20.128.96:5000/nishantchauhan/edc-drool-1:${commitId} ."
                    sh "docker push 172.20.128.96:5000/nishantchauhan/edc-drool-1:${commitId}"
                }    
            }
        }
        stage ('Deploy') {
            container ('helm') {
            
                    sh "/helm init --client-only --skip-refresh"
                    sh "/helm delete --purge nelson"
                    sh "/helm upgrade --debug --install --namespace micro-system --wait --set service.port=80,image.repository=172.20.128.96:5000/nishantchauhan/edc-drool-1,image.tag=${commitId} nelson nelson"
            }    
        }
    }
}
