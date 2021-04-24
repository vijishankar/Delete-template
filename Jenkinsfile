pipeline {

    agent any
    parameters {
        string(name: 'AZSUBSCRIPTION', defaultValue: 'Cloud Technology Solutions', description: 'Azure Subscription Name')
        string(name: 'PROJECT', defaultValue: '', description: 'Project Name')
        choice(name: 'ENVIRONMENT', choices: ['Dev', 'Test', 'stage', 'Uat', 'Sit', 'Prod'], description: 'Environment')
        choice(name: 'LOCATION', choices: ['westeurope', 'northeurope', 'eastus', 'Australia East', 'Australia Southeast'], description: 'Location')

        booleanParam(name: 'Create_CoreResources', defaultValue: 'false', description: '')
        booleanParam(name: 'Create_IngressResources', defaultValue: 'false', description: '')
        booleanParam(name: 'Create_TransactResources', defaultValue: 'false', description: '')
        booleanParam(name: 'All', defaultValue: false, description: 'Deploy all resrouces')
	    
	    
	booleanParam(name: 'Delete', defaultValue: 'false', description: 'Toggel to delete')    
    }
    environment {
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        APP_URL = credentials('APP_URL')
    }

    stages {

        stage('Deploying Resource') {
            steps {
                container('azure') {
                    pwsh ''' 
                            ############################### Powershell #############################
                            
                            $passwd = ConvertTo-SecureString $env:AZURE_CLIENT_SECRET -AsPlainText -Force
                            $pscredential = New-Object System.Management.Automation.PSCredential($env:AZURE_CLIENT_ID, $passwd)
                            Connect-AzAccount -ServicePrincipal -Credential $pscredential -Tenant $env:AZURE_TENANT_ID | Out-null
                            #Write-Output "Azure Subscription is "$Env:AZSUBSCRIPTION""
                            Select-AzSubscription -Subscription "$Env:AZSUBSCRIPTION" | Set-AzContext | Out-null
                            
                            az login --service-principal -u $Env:APP_URL -p $Env:AZURE_CLIENT_SECRET --tenant $Env:AZURE_TENANT_ID | Out-null
                            
                            $Env:SYSTEM_DEFAULTWORKINGDIRECTORY = $env:WORKSPACE
                            $Env:RELEASE_PRIMARYACTIFACTSOURCEALIAS = "/resources"
                            $scriptRoot = $( $Env:SYSTEM_DEFAULTWORKINGDIRECTORY + $Env:RELEASE_PRIMARYACTIFACTSOURCEALIAS )
                            Set-Location $scriptRoot
                            $ErrorActionPreference = "Stop"
                            
                            Write-Output ("============================ Creating Required Variables ============================================")
                            ./DeploymentVars/Set-DeploymentVariables.ps1
                            
                            Write-Output ("======================== Creating Required Resource Groups ===========================================")
                            ./ResourceGroups/New-CoreResourceGroup.ps1
                            ./ResourceGroups/New-IngressvNetResourceGroup.ps1
                            ./ResourceGroups/New-TransactvNetResourceGroup.ps1
                            
                            If ($Env:Create_CoreResources -eq $true)
                            {
                                Write-Output ("============================= Creating Core Resources ===============================================")
                                ./CoreResources/Deploy-CoreResources.ps1
                                ./CoreResources/Deploy-ArtifactStorage.ps1
                                ./CoreResources/Copy-BlobFiles.ps1
                                ./CoreResources/TransferImages.ps1
                            
                            }
                            else
                            {
                                Write-Output ("Create-CoreResources is not selected this is" -f $Env:Create_CoreResources)
                            }
                            If ($Env:Create_IngressResources -eq $true)
                            {
                                Write-Output ("============================= Creating Ingress Resources ===============================================")
                                ./IngressResources/Deploy-Ingressvnet.ps1
                                ./IngressResources/Deploy-Appgateway.ps1
                            }
                            else
                            {
                                Write-Output ("Create-IngressResources is not selected")
                            }
                            If ($Env:Create_TransactResources -eq $true)
                            {
                                Write-Output ("============================= Creating Transact Resources ===============================================")
                                ./TransactResources/Deploy-Network.ps1
                                ./TransactResources/storagePrivateEndpoint.ps1
                                ./TransactResources/Create-ActiveMQ-Fileshare.ps1
                                ./TransactResources/VM/BuildVM.ps1
                                ./TransactResources/aks/Deploy-AKS.ps1
                                ./TransactResources/aks/Set-AgicPermission.ps1
                                ./TransactResources/aks/Create-vNetPeering.ps1
                                ./TransactResources/helmCharts/Install-helmCharts.ps1
                                #./TransactResources/aks/metering-vNet-Integration.ps1
                                #./TransactResources/aks/traceability-vNet-Integration.ps1
                                ./TransactResources/aks/Delete-vNetPeering.ps1
                            }
                            else
                            {
                                Write-Output ("Create-TransactResources is not selected")
                            }
                            if ($Env:All -eq $true)
                            {
                              ./CoreResources/Deploy-CoreResources.ps1
                              ./CoreResources/Deploy-ArtifactStorage.ps1
                              
                              $script1 = './CoreResources/TransferImages.ps1','./CoreResources/Copy-BlobFiles.ps1','./IngressResources/Deploy-Ingressvnet.ps1'
                              $script1 | ForEach-Object -Parallel {
                                Start-Job -FilePath $_
                                Get-Job | Wait-Job | Receive-Job
                              }
                              
                              ./TransactResources/Deploy-Network.ps1
                            
                              $script2 = './IngressResources/Deploy-Appgateway.ps1','./TransactResources/storagePrivateEndpoint.ps1','./TransactResources/aks/Deploy-AKS.ps1','./TransactResources/Create-ActiveMQ-Fileshare.ps1'
                              $script2 | ForEach-Object -Parallel {
                                Start-Job -FilePath $_
                                Get-Job | Wait-Job | Receive-Job
                              }
                              
                              ./TransactResources/aks/Set-AgicPermission.ps1
                              
                              $script3 = './TransactResources/VM/BuildVM.ps1','./TransactResources/aks/Create-vNetPeering.ps1'
                              $script3 | ForEach-Object -Parallel {
                                Start-Job -FilePath $_
                                Get-Job | Wait-Job | Receive-Job
                              }
                              
                              ./TransactResources/helmCharts/Install-helmCharts.ps1
                              ./TransactResources/aks/metering-vNet-Integration.ps1
                              ./TransactResources/aks/traceability-vNet-Integration.ps1
                              ./TransactResources/aks/metering-vNet-Integration.ps1
                              ./TransactResources/aks/traceability-vNet-Integration.ps1
                              ./TransactResources/aks/Delete-vNetPeering.ps1
                            }
                            else {
                              Write-Output ("All is not selected...")
                            }
                '''
                }
            }
        }
    }
    post {
        always {
            echo "\u001B[43;1m Total Build Duration is ${currentBuild.durationString.minus(' and counting')}\u001B[0m"
        }
    }
}
