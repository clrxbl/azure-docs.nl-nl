---
title: Voorbeeld van Azure CLI-script - Schijf van besturingssysteem koppelen | Microsoft Docs
description: Voorbeeld van Azure CLI-script - Schijf van besturingssysteem koppelen
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: bd1bb1dc6222e5abdc03bb9dc5da93751c54da77
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932303"
---
# <a name="troubleshoot-a-vms-operating-system-disk"></a>Problemen met de besturingssysteemschijf van een virtuele machine oplossen

Met dit script koppelt u de schijf van het besturingssysteem van een uitgevallen of problematische virtuele machine als een gegevensschijf aan een tweede virtuele machine. Dit kan nuttig zijn bij het oplossen van schijfproblemen of het herstellen van gegevens.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Voorbeeldscript

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/mount-os-disk/mount-os-disk.sh "Quick Create VM")]

## <a name="script-explanation"></a>Uitleg van het script

In dit script worden de volgende opdrachten gebruikt voor het maken van een resourcegroep, een virtuele machine en alle gerelateerde resources. Elke opdracht in de tabel is een koppeling naar specifieke documentatie over de opdracht.

| Opdracht | Opmerkingen |
|---|---|
| [az vm show](https://docs.microsoft.com/cli/azure/vm#az_vm_show) | Hiermee wordt een lijst met virtuele machines weergegeven. In dit geval wordt de query-optie gebruikt om de schijf met het besturingssysteem van de virtuele machine op te vragen. Deze waarde wordt vervolgens toegevoegd aan de naam van de variabele 'uri'. |
| [az vm delete](https://docs.microsoft.com/cli/azure/vm#az_vm_delete) | Hiermee verwijdert u een virtuele machine. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) | Hiermee maakt u een virtuele machine.  |
| [az vm disk attach](https://docs.microsoft.com/cli/azure/vm/disk#az_vm_disk_attach) | Hiermee wordt een schijf gekoppeld aan een virtuele machine. |
| [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm#az_vm_list_ip_addresses) | Hiermee retourneert u de IP-adressen van een virtuele machine. |

## <a name="next-steps"></a>Volgende stappen

Raadpleeg de [documentatie van Azure CLI](https://docs.microsoft.com/cli/azure) voor meer informatie over de Azure CLI.

U kunt extra CLI-scriptvoorbeelden voor virtuele machines vinden in de [Azure-documentatie voor Linux-VM's](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
