---
title: bestand opnemen
description: bestand opnemen
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 08/07/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 1af4506bc1029efcb6c58ff955087ad0e1311fbc
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/07/2018
ms.locfileid: "44095693"
---
Een [web-app](../articles/app-service/containers/app-service-linux-intro.md) maken in het App Service-plan `myAppServicePlan`. 

In Cloud Shell kunt u de opdracht [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az_webapp_create) gebruiken. Vervang in het volgende voorbeeld `<app_name>` door een unieke naam (geldige tekens zijn `a-z`, `0-9`, en `-`). De runtime is ingesteld op `PYTHON|3.7`. Voer [`az webapp list-runtimes --linux`](/cli/azure/webapp?view=azure-cli-latest#az_webapp_list_runtimes) uit als u alle ondersteunde runtimes wilt zien. 

```azurecli-interactive
# Bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --runtime "PYTHON|3.7" --deployment-local-git
# PowerShell
az --% webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --runtime "PYTHON|3.7" --deployment-local-git
```

Wanneer de web-app is gemaakt, toont de Azure CLI soortgelijke uitvoer als in het volgende voorbeeld:

```json
Local git is configured with url of 'https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

U hebt een nieuwe lege web-app gemaakt, met Git-implementatie ingeschakeld.

> [!NOTE]
> De URL van de externe Git wordt weergegeven in de eigenschap `deploymentLocalGitUrl`, met de indeling `https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git`. Sla deze URL op, want u hebt deze later nodig.
>
