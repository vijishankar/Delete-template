pipeline {

    agent any
    parameters {
        string(name: 'AZSUBSCRIPTION', defaultValue: 'Cloud Technology Solutions', description: 'Azure Subscription Name')
        string(name: 'PROJECT', defaultValue: '', description: 'Project Name')
        choice(name: 'ENVIRONMENT', choices: ['Dev', 'Test', 'stage', 'Uat', 'Sit', 'Prod'], description: 'Environment')
        choice(name: 'LOCATION', choices: ['westeurope', 'northeurope', 'eastus', 'Australia East', 'Australia Southeast'], description: 'Location')

        booleanParam(name: 'Delete', defaultValue: 'false', description: '')
      
    }
    environment {
        AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
        AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
        AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
        APP_URL = credentials('APP_URL')
    }

    stages {
        stage('Deleting Resources') {
            steps {
                container('azure') {
                    pwsh ''' 
                            ############################### Powershell #############################
                            
                            $passwd = ConvertTo-SecureString $env:AZURE_CLIENT_SECRET -AsPlainText -Force
                            $pscredential = New-Object System.Management.Automation.PSCredential($env:AZURE_CLIENT_ID, $passwd)
                            Connect-AzAccount -ServicePrincipal -Credential $pscredential -Tenant $env:AZURE_TENANT_ID | Out-null
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
                            ./ResourceGroups/Delete-IngressvNetResourceGroup.ps1
                            ./ResourceGroups/Delete-TransactvNetResourceGroup.ps1
							./ResourceGroups/Delete-AksResourceGroup.ps1
                            ./ResourceGroups/Delete-CoreResourceGroup.ps1
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
