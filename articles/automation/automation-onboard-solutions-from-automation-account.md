---
title: Meer informatie over het onboarden updatebeheer, wijzigingen bijhouden en inventaris oplossingen in Azure Automation
description: Informatie over hoe zorgen voor onboarding een virtueel Azure-machine met updatebeheer, wijzigingen bijhouden en inventaris oplossingen die deel uitmaken van Azure Automation
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 10/16/2018
ms.topic: conceptual
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: 1ed76b0505747831363ea829f1cb91b3e056282a
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352443"
---
# <a name="onboard-update-management-change-tracking-and-inventory-solutions"></a>Onboarding updatebeheer, wijzigingen bijhouden en inventaris oplossingen

Azure Automation biedt oplossingen voor het beheren van beveiligingsupdates, het bijhouden van wijzigingen en inventaris wat er op uw computers is geïnstalleerd. Er zijn meerdere manieren om Onboarding van machines, u kunt Onboarding van de oplossing [van een virtuele machine](automation-onboard-solutions-from-vm.md), [van meerdere machines Bladeren](automation-onboard-solutions-from-browse.md), via uw Automation-account, of door [runbook](automation-onboard-solutions.md). In dit artikel bevat informatie over onboarding via uw Automation-account.

## <a name="log-in-to-azure"></a>Meld u aan bij Azure.

Aanmelden bij Azure op https://portal.azure.com

## <a name="enable-solutions"></a>Oplossingen inschakelen

Navigeer naar uw Automation-account en selecteer een **voorraad** of **wijzigingen bijhouden** onder **CONFIGURATIEBEHEER**.

Kies de Log analytics-werkruimte en het Automation-account en klikt u op **inschakelen** de oplossing in te schakelen. Het duurt maximaal 15 minuten om de oplossing in te schakelen.

![Onboarding inventarisoplossing](media/automation-onboard-solutions-from-automation-account/onboardsolutions.png)

De oplossing Wijzigingen bijhouden en Inventaris biedt de mogelijkheid om [Wijzigingen bijhouden](automation-vm-change-tracking.md) en [Inventaris](automation-vm-inventory.md) uit te voeren op uw virtuele machines. In deze stap schakelt u de oplossing in op een virtuele machine.

Wanneer wordt gemeld dat de onboarding van de oplossing Wijzigingen bijhouden en Inventarisatie is voltooid, klikt u op **Updatebeheer** onder **CONFIGURATIEBEHEER**.

Met de oplossing Updatebeheer kunt u updates en patches voor uw Azure-VM's beheren. U kunt de status van beschikbare updates beoordelen, de installatie van vereiste updates plannen en de implementatieresultaten bekijken om te controleren of updates correct zijn toegepast op de VM. Deze actie wordt de oplossing voor uw virtuele machine ingeschakeld.

Selecteer **updatebeheer** onder **UPDATEBEHEER**. De geselecteerde Log analytics-werkruimte is dezelfde werkruimte als in de vorige stap. Klik op **Inschakelen** om de onboarding van de oplossing Updatebeheer uit te voeren. Het duurt maximaal 15 minuten om de oplossing in te schakelen.

![Onboarding van update-oplossing](media/automation-onboard-solutions-from-automation-account/onboardsolutions2.png)

## <a name="scope-configuration"></a>Scopeconfiguratie

Elke oplossing maakt gebruik van de configuratie van een bereik in de werkruimte om u te richten op de computers die aan de oplossing. De scopeconfiguratie is een groep van een of meer opgeslagen zoekopdrachten die wordt gebruikt voor het bereik van de oplossing op specifieke computers beperken. Voor toegang tot de bereik-configuraties, in uw Automation-account onder **gerelateerde RESOURCES**, selecteer **werkruimte**. Klik in de werkruimte onder **GEGEVENSBRONNEN voor WERKRUIMTE**, selecteer **Scopeconfiguraties**.

Als de geselecteerde werkruimte beschikt niet over worden de Update Management of bijhouden oplossingen nog, het volgende bereik-configuraties gemaakt:

* **MicrosoftDefaultScopeConfig-voor wijzigingen bijhouden**

* **MicrosoftDefaultScopeConfig-Updates**

Als de geselecteerde werkruimte is al in de oplossing. De oplossing niet opnieuw is geïmplementeerd en configuratie van het bereik is niet toegevoegd aan het.

## <a name="saved-searches"></a>Opgeslagen zoekopdrachten

Wanneer een computer wordt toegevoegd aan de Update Management of de oplossingen wijzigingen bijhouden en inventaris, worden ze toegevoegd aan een van twee opgeslagen zoekopdrachten in uw werkruimte. Deze opgeslagen zoekopdrachten zijn query's die de computers die zijn gericht op deze oplossingen bevatten.

Navigeer naar uw Automation-account en selecteer **opgeslagen zoekopdrachten** onder **algemene**. De twee opgeslagen zoekopdrachten die worden gebruikt door deze oplossingen kunnen worden weergegeven in de volgende tabel:

|Naam     |Categorie  |Alias  |
|---------|---------|---------|
|MicrosoftDefaultComputerGroup     |  Voor wijzigingen bijhouden       | ChangeTracking__MicrosoftDefaultComputerGroup        |
|MicrosoftDefaultComputerGroup     | Updates        | Updates__MicrosoftDefaultComputerGroup         |

Selecteer een opgeslagen zoekopdracht om de query die wordt gebruikt voor het vullen van de groep weer te geven. De volgende afbeelding ziet u de query en de resultaten:

![Opgeslagen zoekopdrachten](media/automation-onboard-solutions-from-automation-account/savedsearch.png)

## <a name="onboard-azure-vms"></a>Azure VM's vrijgeven

Vanuit uw Automation-account selecteren **voorraad** of **bijhouden** onder **CONFIGURATIEBEHEER**, of **updatebeheer** onder **UPDATEBEHEER**.

Klik op **+ Azure-VM's toevoegen**, selecteer een of meer virtuele machines in de lijst. Virtuele machines die kan niet worden ingeschakeld worden grijs weergegeven en kan niet worden geselecteerd. Op de **updatebeheer inschakelen** pagina, klikt u op **inschakelen**. Hiermee wordt de geselecteerde virtuele machines toegevoegd aan de groep van de computer opgeslagen zoekopdracht voor de oplossing.

![Azure-VM's inschakelen](media/automation-onboard-solutions-from-automation-account/enable-azure-vms.png)

## <a name="onboard-a-non-azure-machine"></a>Onboarding van een niet-Azure-machine

Computers niet in Azure moeten handmatig worden toegevoegd. Vanuit uw Automation-account selecteren **voorraad** of **bijhouden** onder **CONFIGURATIEBEHEER**, of **updatebeheer** onder **UPDATEBEHEER**.

Klik op **toevoegen niet-Azure-machine**. Hiermee opent u een nieuw browservenster geopend met de [instructies over het installeren en configureren van de Microsoft Monitoring Agent op de machine](../log-analytics/log-analytics-concept-hybrid.md) zodat de computer kunt beginnen met rapporteren aan de oplossing. Als u onboarding van een machine die momenteel wordt beheerd door System Center Operations Manager, wordt een nieuwe agent is niet vereist, wordt de werkruimtegegevens is ingevoerd in de bestaande agent.

## <a name="onboard-machines-in-the-workspace"></a>Onboarding van machines in de werkruimte

Handmatig geïnstalleerde computers of computers die al rapporteren aan de behoeften van uw werkruimte worden toegevoegd aan Azure Automation voor de oplossing wordt ingeschakeld. Vanuit uw Automation-account selecteren **voorraad** of **bijhouden** onder **CONFIGURATIEBEHEER**, of **updatebeheer** onder **UPDATEBEHEER**.

Selecteer **beheren van virtuele machines**. Hiermee opent u de **Machines beheren** pagina. Deze pagina kunt u de oplossing op een speciale set computers, alle beschikbare machines, inschakelen of de oplossing voor alle huidige machines inschakelen en inschakelen op alle toekomstige computers.

![Opgeslagen zoekopdrachten](media/automation-onboard-solutions-from-automation-account/managemachines.png)

### <a name="all-available-machines"></a>Alle beschikbare machines

Als u de oplossing voor alle beschikbare machines, schakelt **inschakelen op alle beschikbare machines**. Hiermee schakelt het besturingselement machines afzonderlijk toevoegen. Met deze taak wordt de namen van de computers die rapporteren aan de werkruimte aan de groep van de computer opgeslagen zoekquery. Wanneer u hebt geselecteerd, wordt de uitgeschakeld de **Machines beheren** knop.

### <a name="all-available-and-future-machines"></a>Alle beschikbare en toekomstige computers

Als u de oplossing voor alle beschikbare machines en alle toekomstige computers, schakelt **inschakelen op alle beschikbare en toekomstige computers**. Deze optie worden de opgeslagen zoekopdrachten en bereik configuraties uit de werkruimte verwijderd. Hiermee opent u de oplossing voor alle Azure en niet-Azure-machines die aan de werkruimte rapporteren. Wanneer u hebt geselecteerd, wordt de uitgeschakeld de **Machines beheren** knop.

### <a name="selected-machines"></a>Geselecteerde machines

Als u de oplossing voor een of meer machines, schakelt **inschakelen op geselecteerde computers** en klikt u op **toevoegen** naast elke computer die u wilt toevoegen aan de oplossing. Met deze taak wordt de namen van de geselecteerde machines aan de groep van de computer opgeslagen zoekquery voor de oplossing.

## <a name="unlink-workspace"></a>Werkruimte ontkoppelen

De volgende oplossingen zijn afhankelijk van een Log Analytics-werkruimte:

* [Updatebeheer](automation-update-management.md)
* [Tracering wijzigen](automation-change-tracking.md)
* [VM's starten/stoppen buiten kantooruren](automation-solution-vm-management.md)

Als u besluit dat u niet meer wilt integreren van uw Automation-account met Log Analytics, kunt u uw account rechtstreeks vanuit de Azure portal kunt ontkoppelen.  Voordat u doorgaat, moet u eerst te verwijderen van de oplossingen die eerder vermeld, anders wordt dit proces zal worden voorkomen dat u doorgaat. Lees het artikel voor de desbetreffende oplossing die u hebt geïmporteerd voor meer informatie over de stappen die nodig zijn om deze te verwijderen.

Nadat u deze oplossingen hebt verwijderd, kunt u de volgende stappen uit als u wilt loskoppelen van uw Automation-account kunt uitvoeren.

> [!NOTE]
> Sommige oplossingen met inbegrip van eerdere versies van de Azure SQL-oplossing voor controle mogelijk gemaakt automatiseringsactiva en moet mogelijk ook worden verwijderd voordat het ontkoppelen van de werkruimte.

1. Open uw Automation-account vanuit Azure portal, en op de Automation-account selecteren pagina **gekoppelde werkruimte** onder de sectie **gerelateerde Resources** aan de linkerkant.

2. Klik op de pagina van de werkruimte ontkoppelen **werkruimte ontkoppelen**.

   ![Pagina voor werkruimte ontkoppelen](media/automation-onboard-solutions-from-automation-account/automation-unlink-workspace-blade.png).

   U ontvangt een prompt waarin u wordt gevraagd of u wilt doorgaan.

3. Terwijl Azure Automation probeert te ontkoppelen van het account uw Log Analytics-werkruimte, u kunt de voortgang volgen onder **meldingen** in het menu.

Als u de oplossing Update Management gebruikt, kunt indien gewenst u verwijderen van de volgende items die niet meer nodig zijn nadat u de oplossing te verwijderen.

* Plant u update - elk hebben namen die overeenkomen met de update-implementaties die u hebt gemaakt)

* Hybrid worker-groepen gemaakt voor de oplossing - elke naam op dezelfde manier naar machine1.contoso.com_9ceb8108 - 26 c 9-4051-b6b3-227600d715c8).

Als u de VM's starten/stoppen buiten kantooruren oplossing gebruikt, kunt indien gewenst u verwijderen van de volgende items die niet meer nodig zijn nadat u de oplossing te verwijderen.

* Starten en stoppen van VM runbook schema 's
* VM-runbooks starten en stoppen
* Variabelen

## <a name="next-steps"></a>Volgende stappen

Doorgaan met de zelfstudies over de oplossingen voor meer informatie over het gebruik ervan.

* [Zelfstudie - Updates voor uw virtuele machine beheren](automation-tutorial-update-management.md)

* [Zelfstudie - software op een virtuele machine identificeren](automation-tutorial-installed-software.md)

* [Zelfstudie: problemen met wijzigingen op een virtuele machine oplossen](automation-tutorial-troubleshoot-changes.md)