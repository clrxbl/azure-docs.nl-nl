---
title: Azure VM starten is vastgelopen bij Windows Update | Microsoft Docs
description: Meer informatie over het oplossen van het probleem bij het opstarten van een virtuele machine van Azure is vastgelopen bij Windows update.
services: virtual-machines-windows
documentationCenter: ''
authors: genli
manager: cshepard
editor: v-jesits
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/09/2018
ms.author: genli
ms.openlocfilehash: 2d42d2014432b72f35e9b0d9543fe499a6ab721b
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/16/2018
ms.locfileid: "49355444"
---
# <a name="azure-vm-startup-is-stuck-at-windows-update"></a>Azure VM starten is vastgelopen bij Windows update

Dit artikel helpt bij uw virtuele Machine (VM) in de fase van de Windows Update is vastgelopen tijdens het opstarten van het probleem is opgelost. 

> [!NOTE] 
> Azure heeft twee verschillende implementatiemodellen voor het maken van en werken met resources: [Resource Manager en het klassieke model](../../azure-resource-manager/resource-manager-deployment-model.md). In dit artikel bevat informatie over het Resource Manager-implementatiemodel. U wordt aangeraden dat u dit model gebruikt voor nieuwe implementaties in plaats van het klassieke implementatiemodel.

 ## <a name="symptom"></a>Symptoom

 Een Windows-VM niet wordt gestart. Wanneer u de schermafbeeldingen controleren in de [diagnostische gegevens over opstarten](../troubleshooting/boot-diagnostics.md) venster ziet u dat het opstarten van het updateproces is vastgelopen. Hier volgen enkele voorbeelden van berichten die kunnen worden weergegeven:

- Installeren Windows ##% uitschakelen niet in uw PC. Dit duurt even van uw PC wordt meerdere keren opnieuw opstarten
- Houd uw PC op totdat dit is gebeurd. Totaal aantal # installeren bijwerken... 
- De updates die wijzigingen ongedaan maken niet op uw computer uitschakelen kan niet worden voltooid
- Fout bij configureren Windows updates Reverting wijzigingen uitschakelen niet uw computer
- Fout bij < foutcode > het toepassen van update-bewerkingen ### van ### (\Regist...)
- Onherstelbare fout < foutcode > toepassen van update-bewerkingen ### van ### ($...)


## <a name="solution"></a>Oplossing

Afhankelijk van het aantal updates die worden geïnstalleerd of geïmplementeerd back-ups, even kan duren voordat het updateproces. Laat de virtuele machine in deze status voor de 8 uur. Als de virtuele machine nog steeds in deze status na die periode is, start de virtuele machine vanuit Azure portal en zien als deze normaal gesproken kunt starten. Als deze stap niet werkt, probeert u de volgende oplossing.

### <a name="remove-the-update-that-causes-the-problem"></a>Verwijderen van de update die ervoor zorgt het probleem dat

1. Een momentopname van de besturingssysteemschijf van de betrokken VM als een back-up. Zie voor meer informatie, [momentopname maken van een schijf](../windows/snapshot-copy-managed-disk.md). 
2. [De besturingssysteemschijf koppelen aan een virtuele machine voor herstel](troubleshoot-recovery-disks-portal-windows.md).
3. Nadat de besturingssysteemschijf is aangesloten op de virtuele machine voor herstel, opent u de **Schijfbeheer** en zorg ervoor dat het is **ONLINE**. Noteer de stationsletter die is toegewezen aan de gekoppelde besturingssysteemschijf houden van de map \windows. Als de schijf is versleuteld, decoderen van de schijf voordat u doorgaat met de volgende stappen in dit document.

3. Haal de lijst van de updatepakketten die zich op de gekoppelde besturingssysteemschijf:

        dism /image:<Attached OS disk>:\ /get-packages > c:\temp\Patch_level.txt

    Als de gekoppelde besturingssysteemschijf station F, bijvoorbeeld de volgende opdracht uitvoeren:

        dism /image:F:\ /get-packages > c:\temp\Patch_level.txt
4. Open het bestand C:\temp\Patch_level.txt en wordt gelezen uit de onderkant van. Ga naar de update die in **installatie in behandeling** of **in behandeling verwijderen** staat.  Hier volgt een voorbeeld van de status van de update:

     ```
    Package Identity : Package_for_RollupFix~31bf3856ad364e35~amd64~~17134.345.1.5
    State : Install Pending
    Release Type : Security Update
    Install Time :
    ```
5. Verwijder de update die het probleem is veroorzaakt:
    
    ```
    dism /Image:<Attached OS disk>:\ /Remove-Package /PackageName:<PACKAGE NAME TO DELETE>
    ```
    Voorbeeld: 

    ```
    dism /Image:F:\ /Remove-Package /Package_for_RollupFix~31bf3856ad364e35~amd64~~17134.345.1.5
    ```

    > [!NOTE] 
    > De DISM opdrachtregeltool duurt even voor het verwerken van de installatie van de Verenigde Naties, afhankelijk van de grootte van het pakket. Het proces wordt doorgaans binnen 16 minuten worden voltooid.

6. Loskoppelen van de besturingssysteemschijf en vervolgens [opnieuw opbouwen van de virtuele machine met behulp van de besturingssysteemschijf](troubleshoot-recovery-disks-portal-windows.md). 