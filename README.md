# teknologisk

## Table of Content

1. [Logic App Arm Template](#logic-app-arm-template)
1. [Split On](#split-on)
1. [Storage upload](#storage-upload)
1. [Storage Event Grid](#storage-event-grid)
1. [Hybrid Sample](#hybrid-sample)


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


[Back to top](#table-of-content)

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

[Back to top](#table-of-content)

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

## Event Grid Topic

```powershell

##

# TOPIC (Submitte events)

# Subscriptions (Hook on to Topic and subscribe -> WebHook)

$rg = New-AzResourceGroup -Name eventgrid1 -Location 'West Europe'



$eventtopic = New-AzEventGridTopic -ResourceGroupName $rg.ResourceGroupName -Name dtitopic -Location $rg.Location

$key = $eventtopic | Get-AzEventGridTopicKey | select -ExpandProperty Key1

$eventtopic.Endpoint
$key



$endpoint = "https://enenzfrg0eyo.x.pipedream.net/";
New-AzEventGridSubscription -ResourceGroupName $rg.ResourceGroupName -TopicName $eventtopic.TopicName -EventSubscriptionName everythingtorequestbin -Endpoint $endpoint


$endpoint = "https://enl3xauzjznna.x.pipedream.net/";

New-AzEventGridSubscription -ResourceGroupName $rg.ResourceGroupName -TopicName $eventtopic.TopicName -EventSubscriptionName onlyordertorequestbin -Endpoint $endpoint -SubjectBeginsWith 'invoice'

$eventtopic | Get-AzEventGridSubscription | select EventSubscriptionName, Topic

```


EXECUTE TOPIC EVENT

```powershell

Clear-Host
$body = @'
[
{
    "id" : "17",
    "subject" : "order",
    "eventType" : "orderCreated",
    "eventTime" : "2019-06-19T12:00:00.0000Z",
    "data" : {
        "customer" : "Dallas"    
    
    }
}
,

{
    "id" : "18",
    "subject" : "invoice",
    "eventType" : "orderCreated",
    "eventTime" : "2019-06-19T12:00:00.0000Z",
    "data" : {
        "customer" : "Washington"
    }
}
]
'@

curl -Uri $eventtopic.Endpoint -Method Post -Body $body -ContentType "application/json" -Headers @{"aeg-sas-key" = $key}



```

[Back to top](#table-of-content)

## Storage Event Grid

```powershell

$rg = New-AzResourceGroup -Name storageevents -Location westeurope


$storage = New-AzStorageAccount -ResourceGroupName $rg.ResourceGroupName `
    -Name eventgridstorage1239 -SkuName Standard_LRS -Location $rg.Location `
    -Kind StorageV2 -AccessTier Hot


$storage | Get-AzStorageContainer -Name onramp | Get-AzStorageBlob

$storage | Get-AzStorageContainer -Name msgbox | Get-AzStorageBlob



##Upload files

Get-ChildItem -Path C:\Temp\Blobupload | 
    foreach{ $storage | Get-AzStorageContainer -Name onramp | 
    Set-AzStorageBlobContent -File $_.FullName -Force }


##Remove all files

$storage | Get-AzStorageContainer -Name onramp | Get-AzStorageBlob | 
Remove-AzStorageBlob -Force

# $storage | New-AzStorageContainer -Name onramp


## Create Storage Event Subscription
$endpoint = "https://prod-40.westeurope.logic.azure.com:443/workflows/7ecc23a5225744118a451b8c772234c9/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=IsUPSnZIaq06yos-RqnOLQqu62Tv0KXYrZojk-L6JbI";

$storage.Id

New-AzEventGridSubscription -ResourceId $storage.Id -EventSubscriptionName storage2requestbin `
    -Endpoint $endpoint -IncludedEventType "Microsoft.Storage.BlobCreated" `
    -SubjectBeginsWith "/blobServices/default/containers/onramp"


```

## Compose get container and name from subject

```json
"ComposeBlobName": {
                "inputs": "@split(triggerBody()['subject'],'/')[6]",
                "runAfter": {
                    "ComposeBlobPath": [
                        "Succeeded"
                    ]
                },
                "type": "Compose"
            },
            "ComposeBlobPath": {
                "inputs": "@split(triggerBody()['subject'],'/')[4]",
                "runAfter": {},
                "type": "Compose"
            }
```
[Back to top](#table-of-content)


## Hybrid Sample

```csharp

using Microsoft.Azure.Relay;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace HybridService
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {

                MainASync(args).GetAwaiter().GetResult();


                
            }
            catch (Exception ex)
            {

                Console.WriteLine(ex.Message);
            }
        }

        static void SomebodyCalled(RelayedHttpListenerContext context)
        {
            StreamReader sr = new StreamReader(context.Request.InputStream);
            string filePath = sr.ReadToEnd();
            Console.WriteLine($"Somebody called, with the following command: {filePath}");

            var files = Directory.GetFiles(filePath);
            List<object> resultList = new List<object>();
            foreach(string file in files)
            {
                resultList.Add(new { file = file });
            }

            var resultObject = resultList.ToArray();

            var json = JsonConvert.SerializeObject(resultObject, Formatting.Indented);
            context.Response.Headers.Add("content-type", "application/json");
            using(var sw = new StreamWriter(context.Response.OutputStream))
            {
                sw.WriteLine(json);
            }

            context.Response.Close();
        }

        static async Task MainASync(string[] args)
        {
            var listener = GetListener();
            listener.Connecting += (o, e) => { Console.WriteLine("Connecting.."); };
            listener.Online += (o, e) => { Console.WriteLine("Online"); };

            listener.RequestHandler = SomebodyCalled;

            await listener.OpenAsync();

            await Console.In.ReadLineAsync();
        }

        static HybridConnectionListener GetListener()
        {
            string ns = "teknologisk.servicebus.windows.net";
            string hc = "test";

            var listener = new HybridConnectionListener
                (new Uri(String.Format("sb://{0}/{1}", ns, hc)),
                GetTokenProvider());
            return listener;
        }

        static TokenProvider GetTokenProvider()
        {
            string kn = "thelistener";
            string k = "AwRTJ7Mtwm7V+reUgMugfl9pgiggE3/8zpqjUPd8r/I=";
            var result = TokenProvider.CreateSharedAccessSignatureTokenProvider(kn, k);
            return result;
        }
    }
}


```

[Back to top](#table-of-content)
