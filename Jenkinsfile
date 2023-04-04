pipeline{
    agent {
        node{
            label 'master'
        }
    }
    options{
        buildDiscarder(logRotator(numToKeepStr: '5'))
        ansiColor('xterm')
        disableConcurrentBuilds()
    }
    parameters{
        string(name: 'environment', defaultValue: 'a1-apse2', description: '')
        string(name: 'date', defaultValue: '2023-01-12', description: 'Creation date to search. Note that this usually runs 1 day behind current.Format: year-month-day')
        choice(name: 'region', choices: ['-----','us-east-1','us-east-2','us-west-1','us-west-2','ca-central-1','eu-central-1','eu-north-1','eu-west-1','eu-west-2','eu-west-3','ap-south-1','ap-southeast-1','ap-southeast-2','ap-northeast-1','sa-east-1'], description: '')
    }
    stages{
        stage("Checkout"){
            steps{
                script{
                    git credentialsId: 'vghe_jjones',
                        url: 'git@vghe.verint.com:IAAS/harness-clone-automation.git',
                        branch: 'main'
                }
            }
        }
        stage("Get AMI Details"){
            steps{
                script{
                    currentBuild.displayName = "${params.environment} : ${params.date} : ${params.region}"
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: "wfo-clones_jenkins",
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        powershell '''
                            $environment= ${env:environment}
                            $date = ${env:date}
                            $region = ${env:region}
                            .\\powershell\\ami-details.ps1
                        '''
                    }
                }
            }
        }
        stage("Upload to Artifactory"){
            steps{
                script{
                    bat '''
                        dir envFile.properties
                        curl -u "devopstools":"AKCp5cbcrHz3XNdKcxvsavSVhQuBYpUpJYcpgsiHzy2PnzouKwfUnnhM47ZhYzLLGfP2MtnZo" -X PUT "http://10.146.5.213:8082/artifactory/devops/harness/%environment%/%date%/envFile.properties" -T envFile.properties
                    '''
                }
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}
