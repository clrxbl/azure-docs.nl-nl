---
title: Azure Blob-opslaggebeurtenissen doorsturen naar een eindpunt op het aangepaste web - Powershell | Microsoft Docs
description: Gebruik Azure Event Grid om u te abonneren op gebeurtenissen van Blob Storage.
services: storage,event-grid
author: david-stanford
ms.author: dastanfo
ms.date: 08/23/2018
ms.topic: article
ms.service: storage
ms.component: blobs
ms.openlocfilehash: 8482678a9c42fa2d960dee54c9810593cd820553
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/17/2018
ms.locfileid: "45731984"
---
# <a name="route-blob-storage-events-to-a-custom-web-endpoint-with-powershell"></a>Blob-opslaggebeurtenissen doorsturen naar een aangepaste web-eindpunt met PowerShell

Azure Event Grid is een gebeurtenisservice voor de cloud. In dit artikel hebt u Azure PowerShell gebruiken om u te abonneren op gebeurtenissen van Blob storage, trigger een gebeurtenis, en het resultaat weergeven. 

Normaal gesproken verzendt u gebeurtenissen naar een eindpunt dat de gebeurtenisgegevens verwerkt en vervolgens in actie komt. Ter vereenvoudiging van dit artikel stuurt u hier de gebeurtenissen echter naar een web-app die de berichten verzamelt en weergeeft.

Wanneer u klaar bent, ziet u dat de gebeurtenisgegevens naar de web-app zijn verzonden.

![Resultaten weergeven](./media/storage-blob-event-quickstart-powershell/view-results.png)

## <a name="setup"></a>Instellen

Voor dit artikel moet u de nieuwste versie van Azure PowerShell uitvoeren. Zie [Azure PowerShell installeren en configureren](/powershell/azure/install-azurerm-ps) als u de toepassing nog moet installeren of een upgrade moet uitvoeren.

## <a name="log-in-to-azure"></a>Meld u aan bij Azure.

Meld u aan bij uw Azure-abonnement met de opdracht `Connect-AzureRmAccount` en volg de instructies op het scherm.

```powershell
Connect-AzureRmAccount
```

In dit voorbeeld wordt **westus2** en slaat u de selectie in een variabele voor gebruik over de hele.

```powershell
$location = "westus2"
```

## <a name="create-a-resource-group"></a>Een resourcegroep maken

Event Grid-onderwerpen zijn Azure-resources en moeten in een Azure-resourcegroep worden geplaatst. De resourcegroep is een logische verzameling waarin Azure-resources worden geïmplementeerd en beheerd.

Maak een resourcegroep met de opdracht [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup).

In het volgende voorbeeld wordt een resourcegroep met de naam **gridResourceGroup** gemaakt op de locatie **westus2**.  

```powershell
$resourceGroup = "gridResourceGroup"
New-AzureRmResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Create a storage account

Gebeurtenissen van BLOB storage zijn beschikbaar in opslagaccounts voor algemeen gebruik v2 en Blob storage-accounts. **Algemeen gebruik v2** storage-accounts ondersteunen alle functies voor alle storage-services, waaronder Blobs, bestanden, wachtrijen en tabellen. Een **Blob-opslagaccount** is een opslagaccount dat speciaal is bedoeld om ongestructureerde gegevens als blobs (objecten) op te slaan in Azure Storage. Blob-opslagaccounts zijn vergelijkbaar met de opslagaccounts voor algemeen gebruik en bieden dezelfde hoogwaardige kenmerken op het gebied van duurzaamheid, beschikbaarheid, schaalbaarheid en prestaties waarover u nu al beschikt, inclusief 100 procent API-consistentie voor blok-blobs en toevoeg-blobs. Zie voor meer informatie, [overzicht van Azure storage-account](../common/storage-account-overview.md).

Een Blob storage-account maken met het gebruik van LRS-replicatie [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/New-AzureRmStorageAccount), haal vervolgens de opslagaccountcontext op waarin het opslagaccount dat moet worden gebruikt. Als u werkt met een opslagaccount, verwijst u naar de context in plaats van herhaaldelijk de referenties op te geven. Dit voorbeeld maakt u een opslagaccount met de naam **gridstorage** met lokaal redundante opslag (LRS). 

> [!NOTE]
> Namen van opslagaccounts zijn een globale naamruimte, moet u het aantal willekeurige tekens toevoegen aan de naam die in dit script worden opgegeven.

```powershell
$storageName = "gridstorage"
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind BlobStorage `
  -AccessTier Hot

$ctx = $storageAccount.Context
```

## <a name="create-a-message-endpoint"></a>Het eindpunt van een bericht maken

Voordat u zich abonneert op het onderwerp, gaan we het eindpunt voor het gebeurtenisbericht maken. Het eindpunt onderneemt normaal gesproken actie op basis van de gebeurtenisgegevens. Ter vereenvoudiging van deze snelstart gaat u een [vooraf gebouwde web-app](https://github.com/Azure-Samples/azure-event-grid-viewer) implementeren waarmee de gebeurtenisberichten worden weergegeven. De geïmplementeerde oplossing omvat een App Service-plan, een App Service-web-app en broncode van GitHub.

Vervang `<your-site-name>` door een unieke naam voor de web-app. De naam van de web-app moet uniek zijn omdat deze deel uitmaakt van de DNS-vermelding.

```powershell
$sitename="<your-site-name>"

New-AzureRmResourceGroupDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" `
  -siteName $sitename `
  -hostingPlanName viewerhost
```

De implementatie kan enkele minuten duren. Controleer of uw web-app wordt uitgevoerd nadat de implementatie is voltooid. Navigeer in een webbrowser naar: `https://<your-site-name>.azurewebsites.net`

Op de site zouden momenteel geen berichten moeten wijn weergeven.

[!INCLUDE [event-grid-register-provider-powershell.md](../../../includes/event-grid-register-provider-powershell.md)]

## <a name="subscribe-to-your-storage-account"></a>Abonneren op uw opslagaccount

U abonneert u op een onderwerp om Event Grid te laten weten welke gebeurtenissen u wilt traceren. In het volgende voorbeeld ziet u hoe u zich abonneert op het opslagaccount dat u hebt gemaakt, en hoe de URL van de web-app wordt doorgegeven als het eindpunt voor de gebeurtenismelding. Het eindpunt voor uw web-app moet het achtervoegsel `/api/updates/` bevatten.

```powershell
$storageId = (Get-AzureRmStorageAccount -ResourceGroupName $resourceGroup -AccountName $storageName).Id
$endpoint="https://$sitename.azurewebsites.net/api/updates"

New-AzureRmEventGridSubscription `
  -EventSubscriptionName gridBlobQuickStart `
  -Endpoint $endpoint `
  -ResourceId $storageId
```

Bekijk opnieuw uw web-app en u zult zien dat er een validatiegebeurtenis voor een abonnement naartoe is verzonden. Selecteer het oogpictogram om de gebeurtenisgegevens uit te breiden. Via Event Grid wordt de validatiegebeurtenis verzonden zodat het eindpunt kan controleren of de gebeurtenisgegevens in aanmerking komen om ontvangen te worden. De web-app bevat code waarmee het abonnement kan worden gevalideerd.

![Een abonnementgebeurtenis weergeven](./media/storage-blob-event-quickstart-powershell/view-subscription-event.png)

## <a name="trigger-an-event-from-blob-storage"></a>Een gebeurtenis van Blob Storage activeren

Nu gaan we een gebeurtenis activeren om te zien hoe het bericht via Event Grid naar het eindpunt wordt gedistribueerd. Eerst gaan we een container en een object maken. Vervolgens gaan we het object te uploaden naar de container.

```powershell
$containerName = "gridcontainer"
New-AzureStorageContainer -Name $containerName -Context $ctx

echo $null >> gridTestFile.txt

Set-AzureStorageBlobContent -File gridTestFile.txt -Container $containerName -Context $ctx -Blob gridTestFile.txt
```

U hebt de gebeurtenis geactiveerd, en de gebeurtenis is via Event Grid verzonden naar het eindpunt dat u hebt geconfigureerd toen u zich abonneerde. Bekijk uw web-app om de gebeurtenis te zien die u zojuist hebt verzonden.

```json
[{
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Storage/storageAccounts/myblobstorageaccount",
  "subject": "/blobServices/default/containers/gridcontainer/blobs/gridTestFile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-08-16T20:33:51.0595757Z",
  "id": "4d96b1d4-0001-00b3-58ce-16568c064fab",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "Azure-Storage-PowerShell-d65ca2e2-a168-4155-b7a4-2c925c18902f",
    "requestId": "4d96b1d4-0001-00b3-58ce-16568c000000",
    "eTag": "0x8D4E4E61AE038AD",
    "contentType": "application/octet-stream",
    "contentLength": 0,
    "blobType": "BlockBlob",
    "url": "https://myblobstorageaccount.blob.core.windows.net/gridcontainer/gridTestFile.txt",
    "sequencer": "00000000000000EB0000000000046199",
    "storageDiagnostics": {
      "batchId": "dffea416-b46e-4613-ac19-0371c0c5e352"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]

```

## <a name="clean-up-resources"></a>Resources opschonen
Als u verder wilt werken met dit opslagaccount en dit gebeurtenisabonnement, moet u de resources die u hebt gemaakt in dit artikel niet opschonen. Als u niet verder wilt werken, gebruikt u de volgende opdracht om de resources te verwijderen die u in dit artikel hebt gemaakt.

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Volgende stappen

U weet nu hoe u onderwerpen en gebeurtenisabonnementen maakt. Raadpleeg deze onderwerpen voor meer informatie over gebeurtenissen van Blob Storage en waar Event Grid u nog meer bij kan helpen:

- [Reageren op gebeurtenissen van Blob Storage](storage-blob-event-overview.md)
- [Over Event Grid](../../event-grid/overview.md)
