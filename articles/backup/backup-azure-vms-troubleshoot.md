---
title: Back-fouten met virtuele Azure-machine oplossen
description: Problemen oplossen met back-up en herstel van virtuele machines van Azure
services: backup
author: trinadhk
manager: shreeshd
ms.service: backup
ms.topic: conceptual
ms.date: 8/7/2018
ms.author: trinadhk
ms.openlocfilehash: 5dc722b54127731be774bb4a0a7ff9cb359baa76
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/09/2018
ms.locfileid: "42060765"
---
# <a name="troubleshoot-azure-virtual-machine-backup"></a>Problemen oplossen met back-ups van virtuele Azure-machines
U kunt fouten opgetreden tijdens het gebruik van Azure Backup met informatie die worden vermeld in de onderstaande tabel kunt oplossen.

| Foutdetails | Tijdelijke oplossing |
| --- | --- |
| Kan de bewerking niet uitvoeren omdat de VM niet meer bestaat. -Beveiliging van virtuele machine zonder te verwijderen van back-upgegevens stoppen. Meer informatie http://go.microsoft.com/fwlink/?LinkId=808124 |Dit gebeurt wanneer de primaire virtuele machine is verwijderd, maar de back-upbeleid blijft op zoek naar een virtuele machine back-up. Deze fout corrigeren: <ol><li> Opnieuw maken van de virtuele machine met dezelfde naam en dezelfde Resourcegroepnaam [cloudservicenaam]<br>(OF)</li><li> De beveiliging van virtuele machine met of zonder te verwijderen van de back-upgegevens stoppen. [Meer informatie](http://go.microsoft.com/fwlink/?LinkId=808124)</li></ol> |
| De momentopnamebewerking is mislukt omdat er geen netwerkverbinding beschikbaar is op de virtuele machine - Zorg ervoor dat de virtuele machine toegang tot het netwerk heeft. Voor de momentopname zijn voltooid, de Azure-datacenter met geaccepteerde IP-adresbereiken of instellen van een proxyserver voor toegang tot het netwerk. Raadpleeg voor meer informatie, http://go.microsoft.com/fwlink/?LinkId=800034. Als u al proxyserver gebruikt, zorg ervoor dat de instellingen van de proxyserver correct zijn geconfigureerd | Treedt op wanneer u de uitgaande internetverbinding op de virtuele machine weigert. De VM-momentopname-extensie is vereist voor verbinding met internet om een momentopname van de onderliggende schijven. [Zie de sectie over het verhelpen van momentopname mislukt omdat de toegang tot het geblokkeerde netwerk](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#snapshot-operation-failed-due-to-no-network-connectivity-on-the-virtual-machine). |
| VM-agent is kan niet communiceren met de Azure Backup-Service. -Zorg ervoor dat de virtuele machine heeft een netwerkverbinding en de VM-agent is de meest recente en wordt uitgevoerd. Raadpleeg voor meer informatie het artikel http://go.microsoft.com/fwlink/?LinkId=800034. |Deze fout wordt gegenereerd als er een probleem met de VM-Agent of toegang tot het netwerk naar de Azure-infrastructuur is geblokkeerd op een bepaalde manier. [Meer informatie](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#vm-agent-unable-to-communicate-with-azure-backup) over foutopsporing van de virtuele machine een momentopname van problemen.<br> Als de VM-agent is niet wordt veroorzaakt door problemen, start u de virtuele machine opnieuw. Een onjuiste status voor de virtuele machine kan problemen veroorzaken en stelt de status opnieuw starten van de virtuele machine opnieuw. |
| Virtuele machine wordt in een mislukte Inrichtingsstatus - start opnieuw op de virtuele machine en controleer of dat de virtuele machine wordt uitgevoerd of afsluiten. | Deze fout treedt op wanneer een van de extensies fouten VM-status moeten in een mislukte Inrichtingsstatus leidt. Gaat u naar de lijst met extensies en zien of er een extensie is mislukt, verwijdert u deze en probeer het opnieuw opstarten van de virtuele machine. Als alle extensies actief is, controleert u of de VM-agent-service wordt uitgevoerd. Als dat niet het geval is, start de VM-agent-service opnieuw. | 
| Bewerking van VMSnapshot-extensie is mislukt voor beheerde schijven: probeer de back-up opnieuw. Als het probleem zich blijft voordoen, volg de instructies in 'http://go.microsoft.com/fwlink/?LinkId=800034'. Als het probleem zich blijft voordoen, neem dan contact op met Microsoft ondersteuning | Deze fout treedt op wanneer de Backup-service is mislukt voor het activeren van een momentopname. [Meer informatie](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md#vmsnapshot-extension-operation-failed) over foutopsporing op de virtuele machine een momentopname van problemen. |
| Kan niet kopiëren de momentopname van de virtuele machine, vanwege onvoldoende beschikbare ruimte in de storage-account - ervoor te zorgen dat opslagaccount voldoende beschikbare ruimte op de gegevens aanwezig zijn op de premium storage-schijven die zijn gekoppeld aan de virtuele machine bevat. | In het geval van premium VM's op de VM-back-upstack V1 Kopieer we de momentopname naar storage-account. Dit is om ervoor te zorgen dat verkeer van de back-upbeheer, dat op momentopname werkt, heeft geen beperkingen voor het aantal IOP's beschikbaar voor de toepassing met behulp van premium-schijven. Microsoft raadt dat u maximaal 50% (17,5 TB) toewijzen van de totale opslagruimte voor de account, zodat de Azure Backup-service de momentopname naar storage-account en de overdracht van gegevens vanaf deze locatie gekopieerd in storage-account naar de kluis kopiëren kan. | 
| Kan de bewerking niet uitvoeren omdat de VM-agent niet reageert |Deze fout wordt gegenereerd als er een probleem met de VM-Agent of toegang tot het netwerk naar de Azure-infrastructuur is geblokkeerd op een bepaalde manier. Controleer de status van VM-agent-service in services en of de agent wordt weergegeven in het Configuratiescherm voor Windows-VM's. Verwijder het programma van het besturingselement deelvenster en de agent opnieuw te installeren zoals is vermeld [hieronder](#vm-agent). Nadat de agent opnieuw is geïnstalleerd, activeert u een ad-hoc back-up om te controleren. |
| Bewerking voor Recovery services-extensie is mislukt. -Controleer of de meest recente agent van de virtuele machine is aanwezig op de virtuele machine en agent-service wordt uitgevoerd. Voer back-up opnieuw uit. Als de back-upbewerking is mislukt, moet u contact op met Microsoft ondersteuning. |Deze fout wordt gegenereerd wanneer de VM-agent is verouderd. Zie 'De VM-Agent bijwerken' hieronder om bij te werken van de VM-agent. |
| Virtuele machine bestaat niet. -Zorg ervoor dat de virtuele machine bestaat, of Selecteer een andere virtuele machine. |Treedt op wanneer de primaire virtuele machine is verwijderd, maar de back-upbeleid blijft om te zoeken naar een virtuele machine back-up. Deze fout corrigeren: <ol><li> Opnieuw maken van de virtuele machine met dezelfde naam en dezelfde Resourcegroepnaam [cloudservicenaam]<br>(OF)<br></li><li>De beveiliging van de virtuele machine zonder te verwijderen van de back-upgegevens stoppen. [Meer informatie](http://go.microsoft.com/fwlink/?LinkId=808124)</li></ol> |
| Uitvoering van de opdracht is mislukt. -Een andere bewerking wordt momenteel uitgevoerd voor dit item. Wacht totdat de vorige bewerking is voltooid en probeer het vervolgens opnieuw. |Een bestaande back-uptaak wordt uitgevoerd en een nieuwe taak kan niet worden gestart totdat de huidige taak is voltooid. |
| Kopiëren van VHD's van de Recovery Services vault is een time-out: Voer de bewerking binnen een paar minuten opnieuw uit. Neem contact op met Microsoft Ondersteuning als het probleem zich blijft voordoen. | Treedt op als er een tijdelijke fout aan opslag, of als de Backup-service heeft voldoende opslagaccount IOPS gegevens overdraagt naar de kluis binnen de time-outperiode ontvangen. Zorg ervoor dat u de [aanbevolen procedures bij het configureren van uw virtuele machines](backup-azure-vms-introduction.md#best-practices). Uw virtuele machine verplaatsen naar een ander opslagaccount die niet is geladen en probeer de back-uptaak.|
| Back-up is mislukt met een interne fout: probeer de bewerking over een paar minuten opnieuw. Als het probleem zich blijft voordoen, neem dan contact op met Microsoft Support |U ontvangt deze foutmelding voor twee redenen: <ol><li> Er is een tijdelijk probleem bij het openen van de VM-opslag. Controleer de [Azure Status site](https://azure.microsoft.com/status/) om te zien of er Reken-, opslag- of netwerken problemen zijn opgetreden in de regio. Als het probleem opgelost is, probeer de back-uptaak. <li>De oorspronkelijke virtuele machine is verwijderd en het herstelpunt kan niet worden uitgevoerd. De back-upgegevens bewaren voor een verwijderde virtuele machine, maar de back-fouten verwijderen: beveiliging opheffen van de virtuele machine en kies de optie om de gegevens te behouden. Deze actie worden de geplande back-uptaak en de terugkerende foutberichten gestopt. |
| De VM-agent is mislukt voor het installeren van de Azure Recovery Services-extensie op het geselecteerde item - is een vereiste voor de Azure Recovery Services-extensie. De Azure VM-agent installeren en opnieuw starten van de bewerking voor de registratie |<ol> <li>Controleer of de VM-agent correct is geïnstalleerd. <li>Controleer of dat de vlag aan de configuratie van de virtuele machine correct is ingesteld.</ol> [Lees meer](#validating-vm-agent-installation) over het installeren van de VM-agent en het valideren van de installatie van de VM-agent. |
| Installatie van de extensie is mislukt met de fout 'COM + is niet kan communiceren met de Microsoft Distributed Transaction Coordinator |Dit betekent doorgaans dat de service COM + niet wordt uitgevoerd. Neem contact op met Microsoft ondersteuning voor hulp bij het oplossen van dit probleem. |
| De momentopnamebewerking is mislukt met de VSS-bewerkingsfout "dit station is vergrendeld door BitLocker-stationsversleuteling. U moet deze schijf van het Configuratiescherm ontgrendelen. |BitLocker uitschakelen voor alle stations op de virtuele machine en bekijk of de VSS-probleem is opgelost |
| Virtuele machine is niet in een status waarin de back-ups. |<ul><li>Als de virtuele machine een tijdelijke status tussen is **met** en **afsluiten**, wacht totdat de status om te wijzigen en vervolgens de back-uptaak activeert. <li> Als de virtuele machine een Linux-VM is en de module Security Enhanced Linux-kernel gebruikt, sluit u het pad van de Linux-Agent (_/var/lib/waagent_) van het beveiligingsbeleid en zorg ervoor dat de back-up-extensie is geïnstalleerd.  |
| Azure virtuele Machine niet vinden. |Deze fouten treedt op wanneer de primaire virtuele machine is verwijderd, maar de back-upbeleid blijft op zoek naar de verwijderde virtuele machine. Deze fout corrigeren: <ol><li>Opnieuw maken van de virtuele machine met dezelfde naam en dezelfde Resourcegroepnaam [cloudservicenaam] <br>(OF) <li> Schakel de beveiliging voor deze virtuele machine, zodat de back-uptaken, niet worden gemaakt. </ol> |
| VM-agent is niet aanwezig op de virtuele machine: Installeer alle vereiste onderdelen en de VM-agent en start de bewerking vervolgens opnieuw. |[Lees meer](#vm-agent) over de installatie van de VM-agent en het valideren van de installatie van de VM-agent. |
| De momentopnamebewerking is mislukt vanwege VSS-schrijvers in orde |U moet opnieuw opstarten VSS (Volume Shadow copy Service)-schrijvers in orde. Uitvoeren om dit te bereiken, vanaf een opdrachtprompt met verhoogde bevoegdheid, ```vssadmin list writers```. Uitvoer bevat alle VSS-schrijvers en hun status. Voor elke VSS-schrijver waarvan u de status is niet '[1] stabiel' opnieuw starten VSS-schrijver met opdrachten na vanaf een opdrachtprompt met verhoogde bevoegdheid:<br> ```net stop serviceName``` <br> ```net start serviceName```|
| De momentopnamebewerking is mislukt vanwege een fout bij het parseren van de configuratie |Dit gebeurt vanwege een gewijzigde machtigingen op de map als: _%systemdrive%\programdata\microsoft\crypto\rsa\machinekeys_ <br>Voer onderstaande opdracht en Ga na of machtigingen voor map als standaard-één-systemen:<br>_icacls %systemdrive%\programdata\microsoft\crypto\rsa\machinekeys_ <br><br> Standaardmachtigingen zijn:<br>Everyone:(R,W) <br>BUILTIN\Administrators:(F)<br><br>Als u op als directory is anders dan de standaard-machtigingen te zien, volgt u onderstaande stappen op de juiste machtigingen, het verwijderen van het certificaat en het activeren van de back-up.<ol><li>Los machtigingen op als directory.<br>Met behulp van eigenschappen van Verkenner en Advanced Security Settings in de directory, opnieuw instellen van machtigingen terug naar de standaardwaarden. Alle gebruikersobjecten (met uitzondering van standaard) verwijderen uit de map en zorg ervoor dat de **iedereen** machtiging heeft speciale toegang voor:<br>-Map weergeven / gegevens lezen <br>-Kenmerken lezen <br>-Uitgebreide kenmerken lezen <br>-Bestanden maken / gegevens schrijven <br>-Mappen maken / gegevens toevoegen<br>-Kenmerken schrijven<br>-Uitgebreide kenmerken schrijven<br>-Lees-en schrijfmachtigingen<br><br><li>Verwijder alle certificaten die waar **verleend aan** is van het klassieke implementatiemodel of **Windows Azure CRP-Certificaatgenerator**.<ul><li>[Open de console Certificaten (lokale computer)](https://msdn.microsoft.com/library/ms788967(v=vs.110).aspx)<li>Onder **persoonlijke** > **certificaten**, alle certificaten verwijderen waar **verleend aan** is van het klassieke implementatiemodel of **CRP van Windows Azure Certificaat Generator**.</ul><li>Een virtuele machine back-uptaak activeert. </ol>|
| Azure Backup-Service beschikt niet over voldoende machtigingen voor Key Vault voor back-up van versleutelde virtuele Machines. |Backup-service moet worden opgegeven deze machtigingen in PowerShell met behulp van de stappen in **back-up inschakelen** sectie van [PowerShell-documentatie](backup-azure-vms-automation.md). |
|Installatie van een momentopname van extensie is mislukt met fout - de COM + is niet kan communiceren met de Microsoft Distributed Transaction Coordinator | Vanaf een opdrachtprompt met verhoogde bevoegdheid, start u Windows-service COM + System Application, bijvoorbeeld _net start COMSysApp_. <br>Als de service niet kan worden gestart, klikt u vervolgens:<ol><li> Zorg ervoor dat het logboek op het account van de service 'Distributed Transaction Coordinator' is 'Netwerkservice'. Als dat niet het geval is, wijzigt u het logboek voor account 'Netwerkservice', de service opnieuw starten en vervolgens probeert te starten 'COM + System Application'.'<li>Als "COM + System Application wordt niet starten, gebruikt u de volgende stappen uit om service"Distributed Transaction Coordinator"verwijderen/installatie:<br> -De MSDTC-service stoppen<br> -Open een opdrachtprompt (cmd) <br> -Opdracht uitvoeren ```msdtc -uninstall``` <br> -Opdracht uitvoeren ```msdtc -install``` <br> -De MSDTC-service starten<li>Start windows-service COM + System Application'. Eenmaal 'COM + System Application is gestart, trigger een back-uptaak van de Azure-portal.</ol> |
|  De momentopnamebewerking is mislukt vanwege een COM +-fout | De aanbevolen actie is om opnieuw te starten van windows-service COM + System Application (vanaf een opdrachtprompt met verhoogde bevoegdheid - _net start COMSysApp_). Als het probleem zich blijft voordoen, start u de VM opnieuw. Als opnieuw opstarten van de virtuele machine niet wordt opgelost, probeert u [verwijderen van de VMSnapshot-extensie](https://docs.microsoft.com/azure/backup/backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout#cause-3-the-backup-extension-fails-to-update-or-load) en de back-up handmatig activeren. |
| Kan een of meer koppelpunten van de VM voor het maken van een consistente momentopname van de bestandssysteem stilzetten | Voer de volgende stappen uit: <ol><li>Controleer de status van het bestandssysteem van alle gekoppelde apparaten met behulp van _'tune2fs'_ opdracht.<br> Bijvoorbeeld: tune2fs -l/dev/sdb1 \| grep 'Bestandssysteem state' <li>Ontkoppel de apparaten voor welke bestandssysteem status niet met behulp van schone is _'umount'_ opdracht <li> Voer een FileSystemConsistency uit op deze apparaten met behulp van _'fsck'_ opdracht <li> De apparaten opnieuw koppelen en probeer het back-up.</ol> |
| De momentopnamebewerking is mislukt vanwege een fout bij het maken van het communicatiekanaal beveiligd netwerk | <ol><Li> Open de Register-Editor door te voeren regedit.exe in een modus met uitgebreide bevoegdheden. <li> Identificeer alle versies van .NET Framework aanwezig zijn in uw systeem. Ze zijn aanwezig in de hiërarchie van registersleutel "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft" <li> Voor elke .net Framework die aanwezig zijn in de registersleutel, de volgende sleutel toevoegen: <br> "SchUseStrongCrypto"=dword:00000001 </ol>|
| De momentopnamebewerking is mislukt vanwege een fout bij de installatie van Visual C++ Redistributable voor Visual Studio 2012 | Navigeer naar C:\Packages\Plugins\Microsoft.Azure.RecoveryServices.VMSnapshot\agentVersion en vcredist2012_x64 installeren. Zorg ervoor dat de registersleutelwaarde voor het toestaan van de installatie van deze service is ingesteld op de juiste waarde dat wil zeggen de waarde van registersleutel _HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Msiserver_ is ingesteld op 3 en niet-4. Als u nog steeds problemen met de installatie ondervindt, start u de installatieservice opnieuw door te voeren _MSIEXEC/UNREGISTER_ gevolgd door _MSIEXEC /REGISTER_ vanaf een opdrachtprompt met verhoogde bevoegdheid.  |


## <a name="jobs"></a>Taken
| Foutdetails | Tijdelijke oplossing |
| --- | --- |
| Annulering wordt niet ondersteund voor dit taaktype: wacht totdat de taak is voltooid. |Geen |
| De taak is niet in een status is annuleerbaar: wacht totdat de taak is voltooid. <br>OF<br> De geselecteerde taak heeft niet de status is annuleerbaar: wacht tot de taak is voltooid. |De taak is bijna voltooid naar alle waarschijnlijkheid. Wacht tot de taak is voltooid.|
| De taak kan niet worden geannuleerd omdat deze zich niet in voortgang: annulering wordt alleen ondersteund voor taken die uitgevoerd worden. Annuleert u poging op een actieve taak. |Dit gebeurt vanwege een tijdelijke status. Wacht even en probeer de annuleringsbewerking. |
| Kan de taak annuleren: wacht totdat de taak is voltooid. |Geen |

## <a name="restore"></a>Herstellen
| Foutdetails | Tijdelijke oplossing |
| --- | --- |
| Het herstellen is mislukt vanwege een interne Cloud fout |<ol><li>Cloudservice waarop u probeert te herstellen is geconfigureerd met DNS-instellingen. U kunt controleren <br>$deployment = get-Azure-implementatie - servicenaam 'Servicenaam'-sleuf 'Productie' Get-AzureDns - DnsSettings $deployment. DnsSettings<br>Als er is een adres dat is geconfigureerd, betekent dit dat de DNS-instellingen zijn geconfigureerd.<br> <li>Cloudservice waarnaar u wilt herstellen met gereserveerde IP-adres is geconfigureerd en bestaande VM's in de cloudservice wordt gestopt.<br>U kunt controleren op dat een cloudservice IP is gereserveerd met behulp van de volgende powershell-cmdlets:<br>$deployment = get-Azure-implementatie - servicenaam 'servicenaam'-sleuf 'Productie' $dep. ReservedIPName <br><li>U probeert te herstellen van een virtuele machine met de volgende speciale netwerkconfiguraties in dezelfde cloudservice. <br>-Virtuele machines onder de load balancer-configuratie (intern en extern)<br>-Virtuele machines met meerdere gereserveerde IP-adressen<br>-Virtuele machines met meerdere NIC 's<br>Selecteer een nieuwe cloudservice in de gebruikersinterface of Raadpleeg [herstellen overwegingen met betrekking tot](backup-azure-arm-restore-vms.md#restore-vms-with-special-network-configurations) voor virtuele machines met speciale netwerkconfiguraties.</ol> |
| De geselecteerde DNS-naam wordt al gebruikt: Geef een andere DNS-naam en probeer het opnieuw. |De DNS-naam verwijst naar de naam van de cloud-service (gewoonlijk eindigt. cloudapp.net). Dit moet uniek zijn. Als u deze fout optreedt, moet u een andere VM-naam kiezen tijdens het terugzetten. <br><br> Deze fout wordt alleen voor gebruikers van de Azure-portal weergegeven. De herstelbewerking via PowerShell slaagt omdat deze alleen worden hersteld van de schijven en de virtuele machine niet maken. De fout zal worden geconfronteerd bij de virtuele machine expliciet door u gemaakt is nadat de herstelbewerking van de schijf. |
| De configuratie van de opgegeven virtuele netwerk is niet correct: Geef een ander virtueel netwerk-configuratie en probeer het opnieuw. |Geen |
| De opgegeven cloudservice maakt gebruik van een gereserveerde IP-adres, die niet overeenkomt met de configuratie van de virtuele machine die wordt hersteld: Geef een andere cloudservice, die niet van gereserveerde IP-adres gebruikmaakt, of kies een ander herstelpunt kunt herstellen. |Geen |
| Cloudservice is bereikt voor het aantal invoereindpunten: Voer de bewerking opnieuw uit door een andere cloudservice op te geven of met behulp van een bestaand eindpunt. |Geen |
| Recovery Services-kluis en de doelresources storage-account zich in twee verschillende regio's: Controleer of de opslag account opgegeven in de herstelbewerking, bevindt zich in dezelfde Azure-regio als de Recovery Services-kluis. |Geen |
| Opslagaccount dat is opgegeven voor de herstelbewerking niet is ondersteund - alleen Basic/Standard storage-accounts met lokaal redundante of geografisch redundante replicatie-instellingen worden ondersteund. Selecteer een ondersteund opslagaccount |Geen |
| Type Opslagaccount dat is opgegeven voor de herstelbewerking is niet online: Zorg ervoor dat het opslagaccount dat is opgegeven in de herstelbewerking opnieuw online is |Dit kan gebeuren vanwege een tijdelijke fout in Azure Storage of als gevolg van een storing. Kies een ander opslagaccount. |
| Resourcegroep quotum is bereikt: Verwijder enkele resourcegroepen vanuit Azure portal of neem contact op met ondersteuning van Azure om de limieten te verhogen. |Geen |
| Geselecteerde subnet bestaat niet: Selecteer een subnet dat aanwezig is |Geen |
| De Backup-service heeft geen toegang tot resources in uw abonnement. |Om op te lossen dit eerste schijven herstellen met behulp van de stappen in de sectie **terugzetten back-ups van schijven** in [kiezen VM terugzetten configuratie](backup-azure-arm-restore-vms.md#choose-a-vm-restore-configuration). Gebruik daarna de PowerShell-stappen die in [maken van een VM van herstelde schijven](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) volledige VM van herstelde schijven maakt. |

## <a name="backup-or-restore-taking-time"></a>Back-up of herstellen neemt tijd in beslag
Als u ziet uw backup(>12 hours) of restore duurt time(>6 hours):
* Inzicht in [factoren dat bijdraagt aan back-uptijd](backup-azure-vms-introduction.md#total-vm-backup-time) en [factoren die bijdragen aan het herstellen van tijd](backup-azure-vms-introduction.md#total-restore-time).
* Zorg ervoor dat u volgt [back-up van aanbevolen procedures](backup-azure-vms-introduction.md#best-practices). 

## <a name="vm-agent"></a>VM-Agent
### <a name="setting-up-the-vm-agent"></a>Instellen van de VM-Agent
Normaal gesproken is de VM-Agent al aanwezig in virtuele machines die zijn gemaakt op basis van de Azure-galerie. Virtuele machines die zijn gemigreerd vanuit on-premises datacenters hoeft echter niet de VM-Agent is geïnstalleerd. Voor dergelijke VM's moet de VM-Agent expliciet worden geïnstalleerd.

Voor Windows-VM's:

* Download en installeer de [agent-MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). U hebt beheerdersmachtigingen nodig om de installatie te kunnen uitvoeren.
* Voor klassieke virtuele machines, [bijwerken van de VM-eigenschap](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) om aan te geven dat de agent is geïnstalleerd. Deze stap is niet vereist voor virtuele machines Resource Manager.

Voor Linux-VM's:

* Installeer de nieuwste versie van de agent uit de opslagplaats voor distributie. Zie voor meer informatie over de naam van het pakket, de [Linux agent opslagplaats](https://github.com/Azure/WALinuxAgent).
* Voor klassieke virtuele machines, [het blogbericht gebruiken voor het bijwerken van de VM-eigenschap](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx), en verifiëren dat de agent is geïnstalleerd. Deze stap is niet vereist voor virtuele machines Resource Manager.

### <a name="updating-the-vm-agent"></a>De VM-agent bijwerken
Voor Windows-VM's:

* Voor het bijwerken van de VM-Agent opnieuw de [binaire bestanden voor VM-Agent](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Voordat u de agent bijwerkt, zorg er dan voor dat er geen back-upbewerkingen optreden tijdens het bijwerken van de VM-Agent.

Voor Linux-VM's:

* Volg de instructies in het artikel om bij te werken van de Linux VM-Agent [bijwerken van de Linux VM-Agent](../virtual-machines/linux/update-agent.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
**U moet altijd de distributie van-opslagplaats gebruiken om de agent bijwerken**. De agentcode niet downloaden vanuit GitHub. Als de meest recente agent is niet beschikbaar voor uw distributie, kunt u contact opnemen met de ondersteuning voor distributie voor instructies over het verkrijgen van de meest recente agent. U kunt ook controleren op de meest recente [Windows Azure Linux agent](https://github.com/Azure/WALinuxAgent/releases) informatie in de GitHub-opslagplaats.

### <a name="validating-vm-agent-installation"></a>VM-Agent-installatie wordt gevalideerd

Om te controleren of de versie van de VM-Agent op Windows-VM's:

1. Aanmelden bij de virtuele machine van Azure en navigeer naar de map *C:\WindowsAzure\Packages*. Het bestand WaAppAgent.exe moet hier aanwezig zijn.
2. Klik met de rechtermuisknop op het bestand, ga naar **Eigenschappen** en selecteer vervolgens het tabblad **Details**. Het veld productversie moet 2.6.1198.718 of hoger

## <a name="troubleshoot-vm-snapshot-issues"></a>Oplossen van problemen met VM-momentopname
Back-up van virtuele machine is afhankelijk van de van momentopnameopdrachten naar de onderliggende opslag worden uitgegeven. Geen toegang hebben tot opslag- of vertragingen in uitvoering van een momentopname van de taak kan leiden tot de back-uptaak mislukt. De volgende kan leiden tot momentopname taak mislukt.

1. Netwerktoegang tot opslag is geblokkeerd met behulp van NSG<br>
    Meer informatie over het [-netwerktoegang inschakelen](backup-azure-arm-vms-prepare.md#establish-network-connectivity) naar Storage met behulp van een van beide in de Whitelist van IP-adressen of via een proxyserver.
2. Virtuele machines met Sql Server-back-up geconfigureerd kunnen leiden tot de taak voor momentopname vertraagd <br>
   VM-back-up maakt standaard een VSS volledige back-up op Windows-VM's. Virtuele machines waarop SQL Server wordt uitgevoerd, met SQL Server back-up is geconfigureerd, kunnen momentopname vertragingen optreden. Als de momentopname vertragingen in het back-upfouten, stelt u de volgende registersleutel.

   ```
   [HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
   "USEVSSCOPYBACKUP"="TRUE"
   ```

3. VM-status gerapporteerd onjuist, omdat virtuele machine wordt afgesloten in RDP.  <br>
   Als u de extern bureaublad gebruikt om de virtuele machine af te sluiten, of u dat de status van de virtuele machine in de portal juist is. Als de status niet correct is, gebruikt u de **afsluiten** optie in het portaldashboard van de virtuele machine om de virtuele machine af te sluiten.
4. Als meer dan vier virtuele machines dezelfde cloudservice delen, moet u de virtuele machines verdeeld over meerdere back-upbeleid. De back-uptijden dus niet dat meer dan vier VM-back-ups op hetzelfde moment start spreiden. Probeer te scheiden van de begintijden in het beleid door ten minste een uur.
5. Virtuele machine wordt uitgevoerd op veel CPU/geheugen.<br>
   Als de virtuele machine wordt uitgevoerd op veel geheugen of CPU usage(>90%), de taak van de momentopname is in de wachtrij geplaatst, vertraagd en uiteindelijk treedt er een time-out. Als dit gebeurt, kunt u een on-demand back-up.

<br>

## <a name="networking"></a>Netwerken
Net als alle extensies van back-up-extensie toegang nodig tot het openbare internet om te werken. Toegang kunt krijgen tot het openbare internet, kan zich op verschillende manieren manifest:

* De installatie van de extensie kan mislukken
* De back-upbewerkingen (zoals de schijfmomentopname) kunnen mislukken
* Weergeven van de status van de back-upbewerking kan mislukken

De noodzaak voor het oplossen van openbare internet-adressen heeft zijn gelede [hier](http://blogs.msdn.com/b/mast/archive/2014/06/18/azure-vm-provisioning-stuck-on-quot-installing-extensions-on-virtual-machine-quot.aspx). U moet de DNS-configuratie voor het VNET controleren en ervoor te zorgen dat de URI's op Azure opgelost worden kan.

Zodra de naamomzetting correct is uitgevoerd, moet toegang tot de Azure-IP-adressen ook worden opgegeven. Als u wilt deblokkeren toegang tot de Azure-infrastructuur, een van deze stappen te volgen:

1. Het Azure-datacenter geaccepteerde IP-adresbereiken.
   * Lijst van alle [Azure datacenter IP-adressen](https://www.microsoft.com/download/details.aspx?id=41653) goedkeuring.
   * Blokkering van de IP-adressen met behulp van de [New-NetRoute](https://docs.microsoft.com/powershell/module/nettcpip/new-netroute) cmdlet. Voer deze cmdlet uit binnen de Azure-VM in een verhoogde PowerShell-venster (als Administrator uitvoeren).
   * Regels toevoegen aan de NSG (als u voldaan hebt) voor toegang tot de IP-adressen.
2. Maken van een pad voor HTTP-verkeer
   * Als u hebt implementeren sommige netwerkbeperking op locatie (een Netwerkbeveiligingsgroep, bijvoorbeeld) een HTTP-proxy-server om het verkeer te routeren. Stappen voor het implementeren van een HTTP-Proxy-server te vinden [hier](backup-azure-arm-vms-prepare.md#establish-network-connectivity).
   * Regels toevoegen aan de NSG (als u voldaan hebt) voor toegang tot het INTERNET van de HTTP-Proxy.

> [!NOTE]
> DHCP moet zijn ingeschakeld in de Gast voor IaaS VM Backup om te werken.  Als u een statisch privé IP-adres nodig hebt, moet u deze configureren via het platform. De DHCP-optie binnen de virtuele machine moet naar links worden ingeschakeld.
> Meer informatie weergeven over [instellen van een statische interne privé IP-adres](../virtual-network/virtual-networks-reserved-private-ip.md).
>
>
