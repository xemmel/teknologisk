# teknologisk

## Table of Content


## Logic App Arm Template

```json

{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
	"logicAppName" : {
		"type" : "string",
	},
	"location" : {
		"type" : "string",
		"defaultValue" : "[resourceGroup().location]"
	},
	"responseMessage" : {
		"type" : "string",
		"defaultValue" : "Not specified"
	}
  },
  "variables": {
    
  },
  "resources": [
	{
		"name": "[parameters('logicAppName')]",
		"type": "Microsoft.Logic/workflows",
		"location": "[parameters('location')]",
		"apiVersion" : "2016-06-01",
		"properties" : {
			"definition" : {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "Response": {
                "inputs": {
                    "body": "[parameters('responseMessage')]",
                    "statusCode": 200
                },
                "kind": "http",
                "runAfter": {},
                "type": "Response"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {},
        "triggers": {
            "manual": {
                "inputs": {
                    "schema": {},
					"method" : "Get"
                },
                "kind": "Http",
                "type": "Request"
            }
        }
    }
		
		}
	}
  ],
  "outputs": {
	"MortenTest" : {
		"type" : "string",
		"value" : "Hej hej"
	},
	"logicAppUrl": {
      "type": "string",
      "value": "[listCallbackURL(concat(resourceId('Microsoft.Logic/workflows/', parameters('logicAppName')), '/triggers/manual'), '2016-06-01').value]"
   }
  }
}


```

Execute:

```powershell
$deploy_output = New-AzResourceGroupDeployment -ResourceGroupName teknologisk -TemplateFile C:\ARMS\Create_Simple_LogicApp.json -logicAppName template1 -responseMessage "Yes It's working"

```

## XSLT Sample

```xml
<xsl:stylesheet version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:template match="Order">
  <MyOrder>
	<OrderId>
		<xsl:value-of select="ID" />
	</OrderId>
	<ItemNo>
		<xsl:value-of select="Item" />
	</ItemNo>
	<Customer>
		<xsl:text>Coop</xsl:text>
	</Customer>
  </MyOrder>
</xsl:template>

</xsl:stylesheet>


```

### XML INPUT SAMPLE

```xml

<Order>
	<ID>17</ID>
	<Item>Book</Item>
</Order>

```


### Create Logic App from Powershell

```powershell

Clear-Host
$workflowName = "MapXSLT"
$map_name = "xsltmap";

Remove-AzLogicApp -ResourceGroupName $rg_wfmap.ResourceGroupName -Name $workflowName -Force
$defintion = @'
{
    "$schema" : "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json",
    "contentVersion": "1.0.0.0",
    "triggers" : {
        "httpTrigger" : {
            "inputs" : {
                "schema" : {},
                "method" : "Post"
            },
            "type" : "Request",
            "kind" : "Http"
        }
    },
    "actions" : {
        "Transform" : {
            "inputs" : {
                "content" : "@triggerBody()",
                "integrationAccount": {
                    "map" : {
                        "name" : "xsltmap"
                    }
                }
            },
            "type" : "Xslt",
            "runAfter" : {}
        },
        "Response" : {
            "inputs" : {
                "body" : "@outputs('Transform')['body']",
                statuscode: 200
            },
            "type" : "Response",
            "kind" : "Http",
            "runAfter" : {
                "Transform" : ["Succeeded"]   
            }
            
        }
    },
    "outputs" : {
    
    },
    "parameters": {
    
    }
}
'@

New-AzLogicApp -ResourceGroupName $rg_wfmap.ResourceGroupName -Name $workflowName -Location $ia.Location `
    -Definition $defintion -IntegrationAccountId $ia.Id


```


## Split On

POWERSHELL ISE

```powershell

## Create Resource Group
Clear-Host
$rg = New-AzResourceGroup -Name splitit -Location westeurope



## Create Workflow (LA)
Clear-Host
$workflowName = "SplitIt";
Remove-AzLogicApp -ResourceGroupName $rg.ResourceGroupName -Name $workflowName -Force
$defintion = @'
{
    "$schema" : "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json",
    "contentVersion": "1.0.0.0",
    "triggers" : {
        "httpTrigger" : {
            "inputs" : {
                "schema" : {},
                "method" : "Post"
            },
            "type" : "Request",
            "splitOn" : "@triggerBody()",
            "kind" : "Http"
        }
    },
    "actions" : {
        "Compose" : {
            "inputs" : "@triggerBody()",
            "type" : "Compose",
            "runAfter" : {
                
            }
            
        }
    },
    "outputs" : {
    
    },
    "parameters": {
    
    }
}
'@

$la = New-AzLogicApp -ResourceGroupName $rg.ResourceGroupName -Name $workflowName -Location $rg.Location `
    -Definition $defintion

$url = Get-AzLogicAppTriggerCallbackUrl -ResourceGroupName $rg.ResourceGroupName `
     -Name $workflowName -TriggerName httpTrigger | select -ExpandProperty Value

## Call Workflow (LA)
Clear-Host
$body = @'
[
{ "name" : "Morten"},
{ "name" : "la Cour"}

]
'@

curl -Uri $url -Method Post -Body $body -ContentType "application/json"

## Clean Up
Remove-AzResourceGroup -Name $rg.ResourceGroupName -Force 

```

### Storage upload

```powershell

$storage = Get-AzStorageAccount -ResourceGroupName teknologisk -Name teknologiskpre



$storage | Get-AzStorageContainer -Name fromfunction | Get-AzStorageBlob | Remove-AzStorageBlob -Force

## Upload files

$storage | Get-AzStorageContainer -Name tofunction | Set-AzStorageBlobContent -File C:\temp\Blobupload\File1.txt -Force


## List Containers

$storage | Get-AzStorageContainer -Name fromfunction | Get-AzStorageBlob | select Name, Length

$storage | Get-AzStorageContainer -Name tofunction | Get-AzStorageBlob | select Name

```

[Back to top](#table-of-content)
