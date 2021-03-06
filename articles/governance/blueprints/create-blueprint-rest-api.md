---
title: Een Azure Blueprint maken met REST API
description: Gebruik Azure Blueprints om artefacten te maken, definiëren en implementeren.
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: blueprints
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: b873ee869b2044977ebefcfd65331567c24e7ec8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974201"
---
# <a name="define-and-assign-an-azure-blueprint-with-rest-api"></a>Een Azure Blueprint definiëren en toewijzen met REST API

Inzicht in hoe u blauwdrukken in Azure maakt en toewijst, zorgt ervoor dat een organisatie algemene consistentiepatronen kan definiëren en herbruikbare en snel implementeerbare configuraties kan ontwikkelen op basis van Resource Manager-sjablonen, beleid, beveiliging en meer. In deze zelfstudie leert u hoe u Azure Blueprints gebruikt om algemene taken uit te voeren met betrekking tot het maken, publiceren en toewijzen van een blauwdruk binnen uw organisatie, zoals:

> [!div class="checklist"]
> - Een nieuwe blauwdruk maken en verschillende ondersteunde artefacten toevoegen
> - Wijzigingen aanbrengen aan een bestaande blauwdruk die nog een **Concept** is
> - Blauwdrukken markeren als gereed om toe te wijzen met **Gepubliceerd**
> - Een blauwdruk toewijzen aan een bestaand abonnement
> - De status en voortgang van een toegewezen blauwdruk controleren
> - Een blauwdruk verwijderen die aan een abonnement is toegewezen

Als u nog geen abonnement op Azure hebt, maak dan een [gratis account](https://azure.microsoft.com/free) aan voordat u begint.

## <a name="getting-started-with-rest-api"></a>Aan de slag met REST API

Als u niet bekend bent met REST API, begint u met de [Azure REST API-verwijzing](/rest/api/azure/) voor een algemeen begrip van REST API, in het bijzonder van aanvraag-URI en aanvraagbody. In dit artikel worden deze concepten gebruikt om u aan de slag te helpen met Azure Blueprints. Er wordt van uitgegaan dat u er praktische kennis van hebt. Hulpprogramma's zoals [ARMClient](https://github.com/projectkudu/ARMClient) en anderen verwerken autorisatie automatisch en zijn voor beginners aangeraden.

### <a name="rest-api-and-powershell"></a>REST API en PowerShell

Als u nog geen hulpprogramma hebt om REST API-aanroepen te doen, kunt u overwegen PowerShell te gebruiken voor deze instructies. Hieronder volgt een voorbeeldheader voor verificatie met Azure. Genereer een verificatieheader, ook wel een **Bearer-token** genoemd, en zorg ervoor dat de REST API-URI verbinding maakt met parameters of een **Aanvraagbody**:

```powershell-interactive
# Login first with Connect-AzureRmAccount if not using Cloud Shell

$azContext = Get-AzureRmContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2016-06-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Vervang `{subscriptionId}` in de bovenstaande variabele **$restUri** om informatie over uw abonnement op te halen. De variabele $response bevat het resultaat van de cmdlet `Invoke-RestMethod`, die kan worden geparseerd met cmdlets zoals [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Als het service-eindpunt van REST API een **Aanvraagbody** verwacht, geeft u een met JSON geformatteerde variabele op in de parameter `-Body` van `Invoke-RestMethod`.

## <a name="create-a-blueprint"></a>Een blauwdruk maken

De eerste stap bij het definiëren van een standaardpatroon voor naleving is om een blauwdruk samen te stellen uit de beschikbare resources. In dit voorbeeld maakt u een blauwdruk met de naam 'MyBlueprint' om rol- en beleidstoewijzingen voor het abonnement te configureren, een resourcegroep toe te voegen en een Resource Manager-sjabloon en roltoewijzing voor de resourcegroep te maken.

> [!NOTE]
> Wanneer u de REST API gebruikt, wordt het object _blauwdruk_ eerst gemaakt. Voor elk _artefact_ dat wordt toegevoegd en parameters bevat, moeten de parameters vooraf worden gedefinieerd in de eerste _blauwdruk_.

In elke REST API-URI zijn er verschillende variabelen die worden gebruikt en die u moet vervangen door uw eigen waarden:

- Vervang `{YourMG}` door de naam van uw beheergroep
- Vervang `{subscriptionId}` door uw abonnements-ID

1. Maak het eerste _blauwdruk_object. De **Aanvraagbody** bevat eigenschappen van de blauwdruk, te maken resourcegroepen en alle parameters op blauwdrukniveau die tijdens de toewijzing worden ingesteld en worden gebruikt door artefacten die later worden toegevoegd.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

1. Voeg de roltoewijzing toe aan het abonnement. De **Aanvraagbody** definieert het _soort_ artefact en de eigenschappen die zijn afgestemd op de roldefinitie-id en de principal-identiteiten worden doorgegeven als een matrix met waarden. In het onderstaande voorbeeld worden de principal-identiteiten en de opgegeven rol geconfigureerd in een parameter die wordt ingesteld tijdens de toewijzing van de blauwdruk.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleContributor?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

1. Voeg de beleidstoewijzing toe aan het abonnement. De **Aanvraagbody** definieert het _soort_ artefact, de eigenschappen die zijn afgestemd op een beleid of een initiatiefdefinitie en configureert de beleidstoewijzing, zodat deze de blauwdrukparameters definieert die tijdens de toewijzing van de blauwdruk worden gedefinieerd.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyTags?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Voeg nog een beleidstoewijzing toe voor de Storage-tag (gebruik hierbij de parameter _storageAccountType_ opnieuw) aan het abonnement. Deze aanvullende beleidstoewijzingsartefact toont aan dat een parameter die gedefinieerd is in de blauwdruk kan worden gebruikt door meer dan één artefact. In het voorbeeld wordt **storageAccountType** gebruikt om een tag in te stellen in de resourcegroep en wordt informatie opgegeven over het opslagaccount dat in de volgende stap wordt gemaakt.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

1. Voeg een sjabloon toe onder resourcegroep. De **Aanvraagbody** voor een Resource Manager-sjabloon bevat het normale JSON-component van het sjabloon, definieert de doelresourcegroep met **properties.resourceGroup** en gebruikt de blauwdrukparameters **storageAccountType**, **tagName** en **tagValue** opnieuw door elke parameter op te geven in het sjabloon. De blauwdrukparameters worden beschikbaar gemaakt in het sjabloon door **properties.parameters** te definiëren. Binnen de sjabloon-JSON wordt die sleutel/waarde gebruikt om de waarde in te voeren. De namen van de blauwdruk- en sjabloonparameters kunnen dezelfde zijn, maar zijn verschillend gemaakt om aan te tonen hoe elke parameter van de blauwdruk wordt doorgegeven aan de sjabloonartefact.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/templateStorage?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "kind": "template",
         "properties": {
             "template": {
                 "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                 "contentVersion": "1.0.0.0",
                 "parameters": {
                     "storageAccountTypeFromBP": {
                         "type": "string",
                         "defaultValue": "Standard_LRS",
                         "allowedValues": [
                             "Standard_LRS",
                             "Standard_GRS",
                             "Standard_ZRS",
                             "Premium_LRS"
                         ],
                         "metadata": {
                             "description": "Storage Account type"
                         }
                     },
                     "tagNameFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag name from blueprint"
                         }
                     },
                     "tagValueFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag value from blueprint"
                         }
                     }
                 },
                 "variables": {
                     "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
                 },
                 "resources": [{
                     "type": "Microsoft.Storage/storageAccounts",
                     "name": "[variables('storageAccountName')]",
                     "apiVersion": "2016-01-01",
                     "tags": {
                        "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
                     },
                     "location": "[resourceGroup().location]",
                     "sku": {
                         "name": "[parameters('storageAccountTypeFromBP')]"
                     },
                     "kind": "Storage",
                     "properties": {}
                 }],
                 "outputs": {
                     "storageAccountSku": {
                         "type": "string",
                         "value": "[variables('storageAccountName')]"
                     }
                 }
             },
             "resourceGroup": "storageRG",
             "parameters": {
                 "storageAccountTypeFromBP": {
                     "value": "[parameters('storageAccountType')]"
                 },
                 "tagNameFromBP": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValueFromBP": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. Voeg een roltoewijzing toe onder resourcegroep. Net als bij de vorige vermelding van een roltoewijzing wordt in het onderstaande voorbeeld de definitie-id van de rol **Eigenaar** gebruikt en krijgt deze een andere parameter van de blauwdruk.

   - REST API-URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

## <a name="publish-a-blueprint"></a>Een blauwdruk publiceren

Nu de artefacten zijn toegevoegd aan de blauwdruk, is het tijd om deze te publiceren. Als de blauwdruk wordt gepubliceerd, wordt deze toegewezen aan een abonnement.

- REST API-URI

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/versions/{BlueprintVersion}?api-version=2017-11-11-preview
  ```

De waarde voor `{BlueprintVersion}` is een reeks letters, cijfers en afbreekstreepjes (geen spaties of andere speciale tekens) met een maximale lengte van 20 tekens. Gebruik iets unieks en informatiefs als **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Een blauwdruk toewijzen

Nadat een blauwdruk is gepubliceerd met REST API, kan deze worden toegewezen aan een abonnement. Wijs de blauwdruk die u hebt gemaakt toe aan een van de abonnementen in uw beheergroephiërarchie. De **Aanvraagbody** geeft de op te geven blauwdruk op, evenals een naam en locatie voor resourcegroepen in de blauwdrukdefinitie en alle parameters die in de blauwdruk zijn gedefinieerd en die zijn gebruikt door een of meer gekoppelde artefacten.

1. Geef in de Azure Blueprint-service-principal de rol **Eigenaar** op in het doelabonnement. De AppId is statisch (`f71766dc-90d9-4b7d-bd9d-4499c4331c3f`), maar de service-principal-ID verschilt per tenant. U kunt voor uw tenant details aanvragen met de volgende REST API. Deze gebruikt [Azure Active Directory Graph API](../../active-directory/develop/active-directory-graph-api.md), die een andere autorisatie heeft.

   - REST API-URI

     ```http
     GET https://graph.windows.net/{tenantId}/servicePrincipals?api-version=1.6&$filter=appId eq 'f71766dc-90d9-4b7d-bd9d-4499c4331c3f'
     ```

1. Voer de blauwdrukimplementatie uit door deze toe te wijzen aan een abonnement. Als de parameters **inzenders** en **eigenaren** voor de roltoewijzing een matrix van objectIds van de principals nodig hebben, gebruikt u [Azure Active Directory Graph API](../../active-directory/develop/active-directory-graph-api.md) om de objectIds op te halen voor gebruik in de **Aanvraagbody** voor uw eigen gebruikers, groepen of service-principals.

   - REST API-URI

     ```http
     PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2017-11-11-preview
     ```

   - Aanvraagtekst

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

## <a name="unassign-a-blueprint"></a>De toewijzing van een blauwdruk ongedaan maken

Blauwdrukken kunnen uit een abonnement worden verwijderd als deze niet langer nodig zijn of zijn vervangen door nieuwe blauwdrukken met bijgewerkte patronen, ontwerpen en bijgewerkt beleid. Wanneer een blauwdruk wordt verwijderd, worden de artefacten die als onderdeel van die blauwdruk zijn toegewezen, achtergelaten. Als u de toewijzing van een blauwdruk ongedaan wilt maken, gebruikt u de volgende REST API-bewerking:

- REST API-URI

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2017-11-11-preview
  ```

## <a name="delete-a-blueprint"></a>Een blauwdruk verwijderen

Als u de blauwdruk zelf wilt verwijderen, gebruikt u de volgende REST API-bewerking:

- REST API-URI

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2017-11-11-preview
  ```

## <a name="next-steps"></a>Volgende stappen

- Meer informatie over de [levenscyclus van een blauwdruk](./concepts/lifecycle.md)
- Informatie over hoe u [statische en dynamische parameters](./concepts/parameters.md) gebruikt
- Leer hoe u de [blauwdrukvolgorde](./concepts/sequencing-order.md) aanpast
- Ontdek hoe u gebruikmaakt van [resourcevergrendeling in blauwdrukken](./concepts/resource-locking.md)
- Leer hoe u [bestaande toewijzingen bijwerkt](./how-to/update-existing-assignments.md)
- Problemen oplossen tijdens de toewijzing van een blauwdruk met [algemene probleemoplossing](./troubleshoot/general.md)