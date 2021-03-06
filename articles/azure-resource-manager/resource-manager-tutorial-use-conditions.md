---
title: Voorwaarde gebruiken in Azure Resource Manager-sjablonen | Microsoft Docs
description: Leer hoe u Azure-resources implementeert die zijn gebaseerd op voorwaarden.
services: azure-resource-manager
documentationcenter: ''
author: mumian
manager: dougeby
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/02/2018
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 216e474f519e57352b017dc3e6bcdd74d48b03de
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/03/2018
ms.locfileid: "48238643"
---
# <a name="tutorial-use-condition-in-azure-resource-manager-templates"></a>Zelfstudie: Voorwaarde gebruiken in Azure Resource Manager-sjablonen

Leer hoe u Azure-resources implementeert die zijn gebaseerd op voorwaarden. 

Het scenario dat wordt gebruikt in deze zelfstudie is vergelijkbaar met het scenario in [Zelfstudie: Azure Resource Manager-sjablonen met afhankelijke resources maken](./resource-manager-tutorial-create-templates-with-dependent-resources.md). In die zelfstudie hebt u een opslagaccount, een virtuele machine, een virtueel netwerk en enkele andere afhankelijke resources gemaakt. In plaats van een nieuw opslagaccount te maken, laat u mensen kiezen tussen het maken van een nieuw opslagaccount en het gebruiken van een bestaand opslagaccount. Om dit doel te bereiken, definieert u een extra parameter. Als de waarde van de parameter 'new' is, wordt er een nieuw opslagaccount gemaakt.

Deze zelfstudie bestaat uit de volgende taken:

> [!div class="checklist"]
> * Een snelstartsjabloon openen
> * De sjabloon aanpassen
> * De sjabloon implementeren
> * Resources opschonen

Als u geen abonnement op Azure hebt, maakt u een [gratis account](https://azure.microsoft.com/free/) voordat u begint.

## <a name="prerequisites"></a>Vereisten

Als u dit artikel wilt voltooien, hebt u het volgende nodig:

* [Visual Studio Code](https://code.visualstudio.com/) met de [extensie Resource Manager Tools](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)

## <a name="open-a-quickstart-template"></a>Een snelstartsjabloon openen

Azure-snelstartsjablonen is een opslagplaats voor Resource Manager-sjablonen. In plaats van een sjabloon helemaal vanaf de basis te maken, kunt u een voorbeeldsjabloon zoeken en aanpassen. De sjabloon die in deze zelfstudie wordt gebruikt, heet [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/) (Een eenvoudige Windows-VM implementeren).

1. Selecteer in Visual Studio Code **Bestand**>**Bestand openen**.
2. Plak de volgende URL in **Bestandsnaam**:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. Selecteer **Openen** om het bestand te openen.
4. Selecteer **Bestand**>**Opslaan als** om het bestand op uw lokale computer op te slaan als **azuredeploy.json**.

## <a name="modify-the-template"></a>De sjabloon aanpassen

Breng de volgende twee wijzigingen aan in de bestaande sjabloon:

* Voeg een parameter toe die wordt gebruikt om een naam voor het opslagaccount op te geven. Deze parameter geeft gebruikers de mogelijkheid om de naam van een bestaand opslagaccount op te geven. De parameter kan ook worden gebruikt om naam van een nieuw opslagaccount op te geven.
* Voeg een nieuwe parameter toe met de naam **newOrExisting**. De implementatie gebruikt deze parameter om te bepalen waar u er nieuw opslagaccount moet worden gemaakt of dat er een bestaand opslagaccount moet worden gebruikt.

1. Open **azuredeploy.json** in Visual Studio Code.
2. Vervang **variables('storageAccountName')** overal in de sjabloon door **parameters('storageAccountName')**.  **variables('storageAccountName')** komt op drie plaatsen voor.
3. Verwijder de volgende variabeledefinitie:

    ```json
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]",
    ```
4. Voeg de volgende twee parameters toe aan de sjabloon:

    ```json
    "newOrExisting": {
      "type": "string"
    },
    "storageAccountName": {
      "type": "string"
    },
    ```
    De bijgewerkte parameterdefinitie ziet er als volgt uit:

    ![Voorwaarde gebruiken in Resource Manager](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

5. Voeg de volgende regel toe aan het begin van de definitie van het opslagaccount.

    ```json
    "condition": "[equals(parameters('newOrExisting'),'yes')]",
    ```

    De voorwaarde controleert de waarde van een parameter met de naam **newOrExisting**. Als de parameterwaarde **new** is, wordt er een opslagaccount gemaakt.

    De bijgewerkte definitie van het opslagaccount ziet er als volgt uit:

    ![Voorwaarde gebruiken in Resource Manager](./media/resource-manager-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)

6. Sla de wijzigingen op.

## <a name="deploy-the-template"></a>De sjabloon implementeren

Volg de instructies in [De sjabloon implementeren](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template) om de sjabloon te implementeren.

Wanneer u de sjabloon met behulp van Azure PowerShell implementeert, moet u één extra parameter opgeven:

```powershell
$resourceGroupName = "<Enter the resource group name>"
$storageAccountName = "Enter the storage account name>"
$location = "<Enter the Azure location>"
$vmAdmin = "<Enter the admin username>"
$vmPassword = "<Enter the password>"
$dnsLabelPrefix = "<Enter the prefix>"

New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
$vmPW = ConvertTo-SecureString -String $vmPassword -AsPlainText -Force
New-AzureRmResourceGroupDeployment -Name mydeployment0710 -ResourceGroupName $resourceGroupName `
    -TemplateFile azuredeploy.json -adminUsername $vmAdmin -adminPassword $vmPW `
    -dnsLabelPrefix $dnsLabelPrefix -storageAccountName $storageAccountName -newOrExisting "new"
```

> [!NOTE]
> De implementatie mislukt als **newOrExisting** de waarde **new** heeft, maar het opslagaccount met de opgegeven naam al bestaat.

Maak in dat geval een andere implementatie door **newOrExisting** in te stellen op 'existing' en de naam van een bestaand opslagaccount op te geven. Zie [Een opslagaccount maken](../storage/common/storage-quickstart-create-account.md) voor informatie over het vooraf maken van een opslagaccount.

## <a name="clean-up-resources"></a>Resources opschonen

Schoon de geïmplementeerd Azure-resources, wanneer u deze niet meer nodig hebt, op door de resourcegroep te verwijderen.

1. Selecteer **Resourcegroep** in het linkermenu van Azure Portal.
2. Voer de naam van de resourcegroep in het veld **Filter by name** in.
3. Selecteer de naam van de resourcegroep.  U ziet in totaal zes resources in de resourcegroep.
4. Selecteer **Resourcegroep verwijderen** in het bovenste menu.

## <a name="next-steps"></a>Volgende stappen

In deze zelfstudie maakt u een sjabloon waarmee gebruikers kunnen kiezen tussen het maken van een nieuw opslagaccount en het gebruiken van een bestaand opslagaccount. De virtuele machine die u hebt gemaakt in deze zelfstudie vereist een gebruikersnaam en wachtwoord van een beheerder. U kunt in plaats van het wachtwoord door te geven tijdens de implementatie, het wachtwoord met behulp van Azure Key Vault vooraf opslaan en vervolgens ophalen tijdens de implementatie. Zie de volgende zelfstudie voor informatie over het ophalen van geheimen uit Azure Key Vault en hoe u deze geheimen gebruikt in de sjabloonimplementatie:

> [!div class="nextstepaction"]
> [Key Vault integreren in sjabloonimplementatie](./resource-manager-tutorial-use-key-vault.md)
