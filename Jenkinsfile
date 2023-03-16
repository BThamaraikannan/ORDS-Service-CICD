pipeline {
    agent any

    parameters {
        string(defaultValue: 'xx.y.z', 
            description: 'Docker Tag for the ORDS Image to be created, This version should be same as the ORDS App version eg:- 21.4.2', 
            name: 'ORDSTag', 
            trim: false)
	}

    stages {
        stage ('Clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage ('checkout the repo') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/uat_br']], 
                extensions: [], 
                userRemoteConfigs: [[credentialsId: '12b63efe-bcf7-44b3-a11f-a6c3c4faa23e', 
                url: 'https://orahub.oci.oraclecorp.com/csla-cap-dev/ords-cicd.git']]])
            }
        }

        stage ('Build docker Image') {
            steps {
                sh '''
                    #!/bin/bash
                    cp /scratch/jenkins_home/ords*.zip "${WORKSPACE}"/ORDS-Image
                    cd "${WORKSPACE}"/ORDS-Image
                    ls
                    BUILD_START=$(date '+%s')
                    #docker build -t oracle/restdataservices:"${ORDSTag}" .
                '''
            }
        }

        stage ('Push Image to OCI-Registry') {
            steps {
                script {
                    sh "docker tag oracle/restdataservices:${ORDSTag} phx.ocir.io/id7ew7jwukrn/omcs-nonprod/ords:${ORDSTag}"
                    sh "docker login -u 'id7ew7jwukrn/b.thamarai.kannan@oracle.com' -p 'HCBf0ai[0]{}yX_]tmX2' phx.ocir.io"
                    sh "docker push phx.ocir.io/id7ew7jwukrn/omcs-nonprod/ords:${ORDSTag}"
                }
                
            }
        }

        stage('Upload deployment files to kubectl node') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'kubectl-nodes', 
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: 'ls -lrt', execTimeout: 120000, 
                flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, 
                patternSeparator: '[, ]+', remoteDirectory: '', 
                remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.yaml')], 
                usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        
        stage('App deployement to Kubernetes cluster') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'kubectl-node', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'btkannan')]) {
                        def remote = [:]
                        remote.name = 'kubectl-node'
                        remote.host = '140.87.124.184'
                        remote.user = 'btkannan'
                        remote.identityFile = identity
                        remote.allowAnyHosts = true
                        sshCommand remote:remote, command: 'pwd'
                        sshCommand remote:remote, command: 'kubectl config use-context context-ctokamks3ta'
                        sshCommand remote:remote, command: 'kubectl create -f ords_deployment.yaml'
                    }
                }
            }
        }      
    }
}
