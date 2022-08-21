def v_bitBucketUrl = 'https://github.com/shaktipp/SppOtelOperator.git'
def v_bitbucketBranchName = 'develop'
def v_jobName = currentBuild.fullDisplayName


pipeline
{
    agent any

    tools
    {
        //Maven Tool Version
        maven "maven363"
        jdk "jdk18"
    }

    parameters
    {
      choice choices: ['pb-k8s-dev-EKS-Cluster','dms-k8s-dev-EKS-Cluster', 'dms-k8s-ds-EKS-Cluster', 'dms-k8s-qa-EKS-Cluster', 'obs-sandbox-k8s-cluster'], description: 'Selcect Elastic Kubernetes Cluster: ', name: 'eks'
      string defaultValue: '', description: 'Enter Your Access Key', name: 'AWS_ACCESS_KEY_ID'
      string defaultValue: '', description: 'Enter Your Secret Key', name: 'AWS_SECRET_ACCESS_KEY'
      string defaultValue: '', description: 'Enter Your Session Token', name: 'AWS_SESSION_TOKEN'
    }

    stages
    {
        stage('Process Input')
        {
             steps
             {
                 //Jenkins put all the input parameter as params
                 echo "Your Selected EKS Cluster is ${eks}"
                 echo "Access Key is ${params.AWS_ACCESS_KEY_ID}"
                 echo "Secret Key is ${params.AWS_SECRET_ACCESS_KEY}"
                 echo "Session Token is ${params.AWS_SESSION_TOKEN}"
             }
        }

        stage('EKS Connectivity')
        {
            steps
            {
                sh 'export KUBECONFIG=/softwares/dmp/dms-k8s-dev_kubeconfig.txt'
                sh 'unset AWS_DEFAULT_REGION AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN AWS_SECURITY_TOKEN'
                sh 'export AWS_DEFAULT_REGION=us-west-2'
                sh 'export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID'
                sh 'export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY'
                sh 'export AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN'
                sh 'export AWS_SECURITY_TOKEN=$AWS_SESSION_TOKEN'
                echo 'Available Elastic Kubernetes Cluster'
                sh 'aws eks list-clusters --region us-west-2'
                echo 'Switch to ${eks} Cluster'
                sh 'aws eks --region us-west-2 update-kubeconfig --name ${eks} '
                echo 'Find the List of Nodes'
                sh '/home/jenkins/bin/kubectl get nodes'
            }

        }

        stage('Checkout')
        {
            steps
            {
                git credentialsId: 'gitHubCredential', url: v_bitBucketUrl, branch: v_bitbucketBranchName
            }
        }

        stage('Verify NS for Cert Manager')
        {
            steps
            {
                script
                {
                    try
                    {
                        sh 'kubectl create namespace cert-manager'
                    }
                    catch(Error err)
                    {
                        echo "Build Failed as Namespace(cert-manager) already exist, Actual Error: ${err}"
                        currentBuild.result='FAILURE'
                        sh 'exit 1'
                    }
                }//End of Script

            }//End of steps
        }

        stage('Cert Manager')
        {
            steps
            {
                script
                {
                    try
                    {
                        dir("${env.WORKSPACE}/helm/cert-manager")
                        {
                            sh 'kubectl apply -f cert-manager.yaml'
                        }
                    }
                    catch(Error err)
                    {
                        echo "Build Failed due to Error: ${err}"
                        currentBuild.result='FAILURE'
                        sh 'exit 1'
                    }
                }//End of script

            }//End of steps
        }//end of stage Cert Manager

        stage('Verify NS for OTEL Operator')
        {
            steps
            {
                script
                {
                    try
                    {
                        sh 'kubectl create namespace opentelemetry-operator-system'
                    }
                    catch(Error err)
                    {
                        echo "Build Failed as Namespace(opentelemetry-operator-system) already exist, Actual Error: ${err}"
                        currentBuild.result='FAILURE'
                        sh 'exit 1'
                    }
                }//End of Script

            }//End of steps
        }


//         stage('Helm Install Chart')
//         {
//             steps
//             {
//                 dir("${env.WORKSPACE}/helm")
//                 {
//                     echo "Current Directory"
//                     sh "pwd"
//                     echo "Execute Helm Command: helm install jaegerChart --create-namespace --namespace=${namespace} ./jager-chart --wait"
//                     sh 'helm install jaegerChart --create-namespace --namespace=${namespace} ./jager-chart --wait '
//                 }//End of dir block
//             }
//         }//End of stage

    }//End of Stages

}//End of Pipeline