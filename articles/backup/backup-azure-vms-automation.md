---
title: Met PowerShell back-ups implementeren en beheren voor door Resource Manager geïmplementeerde virtuele machines
description: PowerShell gebruiken om te implementeren en beheren van back-ups in Azure voor Resource Manager geïmplementeerde virtuele machines
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 10/20/2018
ms.author: markgal
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c29a91a40df34ecd9270d5805209d361cf990754
ms.sourcegitcommit: 17633e545a3d03018d3a218ae6a3e4338a92450d
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/22/2018
ms.locfileid: "49638032"
---
# <a name="use-powershell-to-back-up-and-restore-virtual-machines"></a>PowerShell gebruiken voor back-up en herstellen van virtuele machines

In dit artikel laat zien hoe u Azure PowerShell-cmdlets voor back-up en herstel van een Azure virtuele machine (VM) van een Recovery Services-kluis. Een Recovery Services-kluis is een Azure Resource Manager-resource die is gebruikt voor het beveiligen van gegevens en assets in Azure Backup en Azure Site Recovery-services. 

> [!NOTE]
> Azure heeft twee implementatiemodellen voor het maken van en werken met resources: [Resource Manager en het klassieke model](../azure-resource-manager/resource-manager-deployment-model.md). In dit artikel is bedoeld voor gebruik met virtuele machines die zijn gemaakt met behulp van de Resource Manager-model.
>
>

In dit artikel begeleidt u bij het beveiligen van een virtuele machine en gegevens herstellen vanaf een herstelpunt met behulp van PowerShell.

## <a name="concepts"></a>Concepten
Als u niet bekend met de Azure Backup-service, voor een overzicht van de service bent, Zie het artikel [wat is Azure Backup?](backup-introduction-to-azure-backup.md) Voordat u begint, zorg ervoor dat u de vereisten die nodig zijn met Azure Backup en de beperkingen van de huidige VM-back-upoplossing dekt.

Voor het gebruik van PowerShell effectief, is het nodig om inzicht in de hiërarchie van objecten en van waar u wilt beginnen.

![Recovery Services-objecthiërarchie](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Voor de naslaginformatie over AzureRm.RecoveryServices.Backup PowerShell-cmdlets, raadpleegt u de [Azure Backup - Recovery Services-Cmdlets](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup) in de Azure-bibliotheek.

## <a name="setup-and-registration"></a>Installatie en registratie

Om te beginnen met:

1. [Download de nieuwste versie van PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) (de minimaal vereiste versie is: 1.4.0)

2. De beschikbare Azure Backup PowerShell-cmdlets vinden door de volgende opdracht te typen:
   
    ```powershell
    Get-Command *azurermrecoveryservices*
    ```    
    De aliassen en cmdlets voor back-up van Azure, Azure Site Recovery en de Recovery Services-kluis worden weergegeven. De volgende afbeelding is een voorbeeld van wat u ziet. Het is niet de volledige lijst met cmdlets.

    ![lijst met Recovery Services](./media/backup-azure-vms-automation/list-of-recoveryservices-ps.png)

3. Aanmelden bij uw Azure-account met **Connect-AzureRmAccount**. Deze cmdlet wordt een webpagina vraagt u om referenties voor uw account:

    * U kunt ook uw accountreferenties opnemen als een parameter in de **Connect-AzureRmAccount** cmdlet, met behulp van de **-referentie** parameter.
    * Als u de CSP-partner werken namens een tenant bent, geeft u de klant als een tenant met behulp van de naam van de primaire domeincontroller tenant-id of tenant. Bijvoorbeeld: **Connect-AzureRmAccount-Tenant "fabrikam.com"**

4. Koppel het abonnement dat u gebruiken met het account wilt, omdat een account kan meerdere abonnementen hebben:

    ```powershell
    Select-AzureRmSubscription -SubscriptionName $SubscriptionName
    ```

5. Als u Azure Backup voor de eerste keer gebruikt, moet u de **[Register-AzureRmResourceProvider](http://docs.microsoft.com/powershell/module/azurerm.resources/register-azurermresourceprovider)** cmdlet voor het registreren van de Azure Recovery Service-provider met uw abonnement.

    ```powershell
    Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. U kunt controleren of de Providers is geregistreerd met de volgende opdrachten:
    ```powershell
    Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ``` 
    In de uitvoer van de opdracht, de **RegistrationState** moet wijzigen naar **geregistreerde**. Als niet het geval is, alleen wordt uitgevoerd de **[Register-AzureRmResourceProvider](http://docs.microsoft.com/powershell/module/azurerm.resources/register-azurermresourceprovider)** cmdlet opnieuw uit.

De volgende taken kunnen worden geautomatiseerd met PowerShell:

* [Een Recovery Services-kluis maken](backup-azure-vms-automation.md#create-a-recovery-services-vault)
* [Back-ups maken van Azure-VM's](backup-azure-vms-automation.md#back-up-azure-vms)
* [Een back-uptaak activeert](backup-azure-vms-automation.md#trigger-a-backup-job)
* [Een back-uptaak controleren](backup-azure-vms-automation.md#monitoring-a-backup-job)
* [Een Azure-VM herstellen](backup-azure-vms-automation.md#restore-an-azure-vm)

## <a name="create-a-recovery-services-vault"></a>Een Recovery Services-kluis maken

De volgende stappen leiden u bij het maken van een Recovery Services-kluis. Een Recovery Services-kluis is anders dan een back-upkluis.

1. De Recovery Services-kluis is een Resource Manager-resource, dus u moet dit binnen een resourcegroep te plaatsen. U kunt een bestaande resourcegroep gebruiken, of maak een resourcegroep met de **[New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup)** cmdlet. Bij het maken van een resourcegroep, geef de naam en locatie voor de resourcegroep.  

    ```powershell
    New-AzureRmResourceGroup -Name "test-rg" -Location "West US"
    ```
2. Gebruik de **[New-AzureRmRecoveryServicesVault](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices/new-azurermrecoveryservicesvault)** cmdlet voor het maken van de Recovery Services-kluis. Zorg ervoor dat de dezelfde locatie voor de kluis opgeven dat werd gebruikt voor de resourcegroep.

    ```powershell
    New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```
3. Geef het type opslagredundantie moet gebruiken. u kunt [lokaal redundante opslag (LRS)](../storage/common/storage-redundancy-lrs.md) of [geografisch redundante opslag (GRS)](../storage/common/storage-redundancy-grs.md). Het volgende voorbeeld ziet dat de optie - BackupStorageRedundancy voor testvault is ingesteld op GeoRedundant.

    ```powershell
    $vault1 = Get-AzureRmRecoveryServicesVault -Name "testvault"
    Set-AzureRmRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

   > [!TIP]
   > Voor veel Azure Backup-cmdlets is het object Recovery Services-kluis als invoer vereist. Daarom is het handiger het object Backup Recovery Services-kluis in een variabele op te slaan.
   >
   >

## <a name="view-the-vaults-in-a-subscription"></a>De kluizen in een abonnement weergeven

U kunt alle kluizen weergeven in het abonnement, met  **[Get-AzureRmRecoveryServicesVault](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices/get-azurermrecoveryservicesvault)**:

```powershell
Get-AzureRmRecoveryServicesVault
```

De uitvoer is vergelijkbaar met het volgende voorbeeld, ziet u de dat locatie en de bijbehorende ResourceGroupName worden geleverd.

```
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```


## <a name="back-up-azure-vms"></a>Back-ups maken van Azure-VM's

Een Recovery Services-kluis gebruiken ter bescherming van uw virtuele machines. Voordat u de beveiliging toepassen, stelt de context van de kluis (het type van de gegevens die worden beveiligd in de kluis) en controleer of het beveiligingsbeleid. Het beveiligingsbeleid is het schema voor de back-uptaken uitgevoerd en hoelang elke back-upmomentopname wordt bewaard.

### <a name="set-vault-context"></a>De context van de kluis instellen

Voordat u de beveiliging op een virtuele machine inschakelt, gebruikt u **[Set-AzureRmRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices/set-azurermrecoveryservicesvaultcontext)** om in te stellen van de context van de kluis. Zodra deze is ingesteld, is deze op alle navolgende cmdlets van toepassing. Het volgende voorbeeld wordt de context van de kluis voor de kluis *testvault*.

```powershell
Get-AzureRmRecoveryServicesVault -Name "testvault" | Set-AzureRmRecoveryServicesVaultContext
```

### <a name="create-a-protection-policy"></a>Een beveiligingsbeleid maken

Als u een Recovery Services-kluis maakt, gaat deze gepaard met standaardbeleid voor beveiliging en retentie. Volgens het standaardbeveiligingsbeleid wordt elke dag op een bepaald tijdstip een back-uptaak getriggerd. Volgens het standaardbewaarbeleid wordt het dagelijkse herstelpunt gedurende dertig dagen bewaard. Het standaardbeleid kunt u snel uw VM te beveiligen en het beleid later met verschillende gegevens te bewerken.

Gebruik **[Get-AzureRmRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupprotectionpolicy)** om het beveiligingsbeleid voor apps beschikbaar in de kluis weer te geven. U kunt deze cmdlet gebruiken om op te halen van een specifiek beleid, of om de beleidsregels die zijn gekoppeld aan een type werkbelasting weer te geven. Het volgende voorbeeld wordt een beleid voor het type werkbelasting, installatiekopieën van virtuele machines.

```powershell
Get-AzureRmRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM"
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM
```

> [!NOTE]
> De tijdzone van het veld BackupTime in PowerShell is UTC. Echter, wanneer de back-uptijd wordt weergegeven in de Azure-portal, de tijd wordt aangepast aan uw lokale tijdzone.
>
>

Een back-protection-beleid is gekoppeld aan ten minste één beleid voor Gegevensretentie. Bewaarbeleid wordt gedefinieerd hoe lang een herstelpunt wordt bewaard voordat deze wordt verwijderd. Gebruik **[Get-AzureRmRecoveryServicesBackupRetentionPolicyObject](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupretentionpolicyobject)** om het standaardretentiebeleid weer te geven.  Op deze manier kunt u **[Get-AzureRmRecoveryServicesBackupSchedulePolicyObject](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupschedulepolicyobject)** om op te halen van het standaardbeleid voor planning. De **[New-AzureRmRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/new-azurermrecoveryservicesbackupprotectionpolicy)** met de cmdlet maakt een PowerShell-object dat back-upbeleid informatie bevat. De beleidsobjecten schema en de retentie worden gebruikt als invoer voor de **[New-AzureRmRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/new-azurermrecoveryservicesbackupprotectionpolicy)** cmdlet. Het volgende voorbeeld wordt het beleid voor planning en het bewaarbeleid in variabelen. Het voorbeeld deze variabelen worden gebruikt voor het definiëren van de parameters bij het maken van een beveiligingsbeleid *NewPolicy*.

```powershell
$schPol = Get-AzureRmRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
New-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType "AzureVM" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM
```

### <a name="enable-protection"></a>Beveiliging inschakelen

Als u het beveiligingsbeleid hebt gedefinieerd, moet u nog steeds het beleid voor een item inschakelen. Gebruik **[Enable-AzureRmRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/enable-azurermrecoveryservicesbackupprotection)** beveiliging in te schakelen. Inschakelen van beveiliging vereist twee objecten - het item en het beleid. Nadat het beleid gekoppeld aan de kluis is, worden de back-werkstroom wordt geactiveerd op het moment dat is gedefinieerd in de planning van het beleid.

De volgende voorbeelden inschakelen van de beveiliging voor het item, V2VM, met behulp van het beleid, NewPolicy. De voorbeelden zijn afhankelijk van of de virtuele machine is versleuteld en wat het type versleuteling. 

De beveiliging inschakelen op **resourcemanager-VM's niet-versleutelde**:

```powershell
$pol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

De beveiliging inschakelen op **versleutelde virtuele machines (versleuteld met behulp van (bek) en KEK)**, moet u de Azure Backup-service-machtiging voor het lezen van sleutels en geheimen van de key vault geven.

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

De beveiliging inschakelen op **versleutelde virtuele machines (versleuteld met behulp van de BEK alleen)**, moet u de Azure Backup-service-machtiging voor het lezen van geheimen van de key vault geven.

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToSecrets backup,get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"
```

> [!NOTE]
> Als u van de Azure Government-cloud gebruikmaakt, gebruikt u de ff281ffe-705c-4f53-9f37-a40e6f2c68f3 waarde voor de parameter **- ServicePrincipalName** in [Set-AzureRmKeyVaultAccessPolicy](https://docs.microsoft.com/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) cmdlet.
>
>

Beveiliging in te schakelen op een klassieke virtuele machine:

```powershell
$pol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V1VM" -ServiceName "ServiceName1"
```

### <a name="modify-a-protection-policy"></a>Een beveiligingsbeleid wijzigen

Gebruiken voor het wijzigen van het beveiligingsbeleid, [Set-AzureRmRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/set-azurermrecoveryservicesbackupprotectionpolicy) de SchedulePolicy of RetentionPolicy objecten te wijzigen.

Het volgende voorbeeld wordt de bewaarperiode voor herstelpunten en 365 dagen.

```powershell
$retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
$retPol.DailySchedule.DurationCountInDays = 365
$pol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
Set-AzureRmRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy $RetPol
```

## <a name="trigger-a-backup"></a>Een back-up activeren

Gebruik **[Backup-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/backup-azurermrecoveryservicesbackupitem)** voor het activeren van een back-uptaak. Als dit de eerste back-up is, is een volledige back-up. Volgende back-ups duren voordat een incrementele kopie. Zorg ervoor dat u **[Set-AzureRmRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices/set-azurermrecoveryservicesvaultcontext)** om in te stellen van de context van de kluis voordat de back-uptaak wordt geactiveerd. Het volgende voorbeeld wordt ervan uitgegaan dat de context van de kluis is al ingesteld.

```powershell
$namedContainer = Get-AzureRmRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$item = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM"
$job = Backup-AzureRmRecoveryServicesBackupItem -Item $item
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM              Backup              InProgress          4/23/2016                  5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

> [!NOTE]
> De tijdzone van de StartTime en EndTime velden in PowerShell is UTC. Echter, wanneer de tijd wordt weergegeven in de Azure-portal, de tijd wordt aangepast aan uw lokale tijdzone.
>
>

## <a name="monitoring-a-backup-job"></a>Bewaking van een back-uptaak

U kunt langlopende bewerkingen, zoals back-uptaken controleren zonder gebruik van de Azure-portal. Als u de status van een taak wordt uitgevoerd, gebruikt de **[Get-AzureRmRecoveryservicesBackupJob](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupjob)** cmdlet. Deze cmdlet haalt de back-uptaken voor een bepaalde kluis en die vault is opgegeven in de context van de kluis. Het volgende voorbeeld wordt de status van een taak wordt uitgevoerd als een matrix en slaat de status in de $joblist-variabele.

```powershell
$joblist = Get-AzureRmRecoveryservicesBackupJob –Status "InProgress"
$joblist[0]
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM             Backup               InProgress            4/23/2016                5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

In plaats van deze taken voor de voltooiing - dit is niet nodig als u meer code - polling gebruiken de **[Wait-AzureRmRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/wait-azurermrecoveryservicesbackupjob)** cmdlet. Deze cmdlet wordt de uitvoering onderbroken totdat de taak is voltooid of de opgegeven time-outwaarde is bereikt.

```powershell
Wait-AzureRmRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200
```

## <a name="restore-an-azure-vm"></a>Een Azure-VM herstellen

Er is een belangrijk verschil tussen het herstellen van een virtuele machine met behulp van de Azure portal en herstellen van een virtuele machine met behulp van PowerShell. Met PowerShell, is de herstelbewerking is voltooid nadat de schijven en configuratie-informatie van het herstelpunt worden gemaakt. De herstelbewerking maken niet de virtuele machine. Voor het maken van een virtuele machine van de schijf, Zie de sectie [maken van de VM van herstelde schijven](backup-azure-vms-automation.md#create-a-vm-from-restored-disks). Als u niet wilt dat de volledige VM herstellen, maar u wilt herstellen of een aantal bestanden herstellen van een Azure-VM back-up, raadpleegt u de [sectie herstel het bestand](backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup).

> [!Tip]
> De herstelbewerking wordt de virtuele machine niet maken.
>
>

De volgende afbeelding toont de objecthiërarchie van de RecoveryServicesVault omlaag naar de BackupRecoveryPoint.

![Recovery Services-objecthiërarchie BackupContainer weergeven](./media/backup-azure-vms-arm-automation/backuprecoverypoint-only.png)

Voor het herstellen van back-upgegevens, Identificeer het item waarvan een back-up is gemaakt en het herstelpunt die de point-in-time-gegevens bevat. Gebruik **[Restore-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/restore-azurermrecoveryservicesbackupitem)** voor het terugzetten van gegevens uit de kluis aan uw account.

De basisstappen voor het herstellen van een virtuele machine van Azure zijn:

* Selecteer de VM.
* Kies een herstelpunt.
* Herstel de schijven.
* De virtuele machine maken van opgeslagen schijven.

### <a name="select-the-vm"></a>Selecteer de virtuele machine

Als u de PowerShell-object waarmee de juiste back-artikel, starten van de container in de kluis en uw eigen manier omlaag in de objecthiërarchie werken. Selecteer de container die staat voor de virtuele machine, gebruikt u de **[Get-AzureRmRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupcontainer)** cmdlet en doorgeven die u wilt de **[ Get-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupitem)** cmdlet.

```powershell
$namedContainer = Get-AzureRmRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$backupitem = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM"
```

### <a name="choose-a-recovery-point"></a>Kies een herstelpunt

Gebruik de **[Get-AzureRmRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackuprecoverypoint)** cmdlet om alle herstelpunten voor de back-upitem weer te geven. Kies vervolgens het herstelpunt te herstellen. Als u niet zeker welk herstelpunt weet te gebruiken, is het raadzaam om te kiezen van de meest recente RecoveryPointType = AppConsistent punt in de lijst.

In het volgende script, de variabele **$rp**, is een matrix van herstelpunten voor de geselecteerde back-upitem van de afgelopen zeven dagen. De matrix is in omgekeerde volgorde gesorteerd op tijd wordt opgelost met de meest recente herstelpunt bij index 0. Gebruik de standaard PowerShell matrix indexeren om op te halen van het herstelpunt. In het voorbeeld selecteert $rp [0] het meest recente herstelpunt.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzureRmRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0]
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="restore-the-disks"></a>De schijven herstellen

Gebruik de **[Restore-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/restore-azurermrecoveryservicesbackupitem)** cmdlet om te herstellen van gegevens en configuratie van een back-upitem naar een herstelpunt. Wanneer u een herstelpunt hebt geïdentificeerd, gebruikt u deze als de waarde voor de **- RecoveryPoint** parameter. In het bovenstaande voorbeeld **$rp [0]** is van het herstelpunt dat u wilt gebruiken. In de volgende voorbeeldcode **$rp [0]** is van het herstelpunt dat u wilt gebruiken voor het herstellen van de schijf.

De schijven en configuratie-informatie terugzetten:

```powershell
$restorejob = Restore-AzureRmRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG"
$restorejob
```
#### <a name="restore-managed-disks"></a>Beheerde schijven terugzetten

> [!NOTE]
> Als de back-ups virtuele machine schijven beheerde er en u wilt deze als beheerde schijven terugzetten, hebben we de mogelijkheid van Azure Powershell v 6.7.0 geïntroduceerd. en hoger
>
>

Geef een extra parameter **TargetResourceGroupName** om op te geven van de RG waarmee beheerde schijven wordt hersteld.


```powershell
$restorejob = Restore-AzureRmRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks"
```

De **VMConfig.JSON** bestand wordt hersteld naar het opslagaccount en de beheerde schijven worden hersteld met het opgegeven doel-RG.


De uitvoer lijkt op die in het volgende voorbeeld:
```
WorkloadName     Operation          Status               StartTime                 EndTime            JobID
------------     ---------          ------               ---------                 -------          ----------
V2VM              Restore           InProgress           4/23/2016 5:00:30 PM                        cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Gebruik de **[Wait-AzureRmRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/wait-azurermrecoveryservicesbackupjob)** cmdlet na afloop van de hersteltaak om te voltooien.

```powershell
Wait-AzureRmRecoveryServicesBackupJob -Job $restorejob -Timeout 43200
```

Als de hersteltaak is voltooid, gebruikt u de **[Get-AzureRmRecoveryServicesBackupJobDetails](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupjobdetails)** cmdlet om op te halen van de details van de herstelbewerking opnieuw. De eigenschap JobDetails heeft de informatie die nodig zijn voor de virtuele machine opnieuw.

```powershell
$restorejob = Get-AzureRmRecoveryServicesBackupJob -Job $restorejob
$details = Get-AzureRmRecoveryServicesBackupJobDetails -Job $restorejob
```

Nadat u de schijven herstellen, gaat u naar de volgende sectie om de VM te maken.

## <a name="create-a-vm-from-restored-disks"></a>Een virtuele machine maken van herstelde schijven

Nadat u de schijven herstellen, gebruikt u de volgende stappen uit te maken en configureren van de virtuele machine van de schijf.

> [!NOTE]
> Voor het maken van versleutelde virtuele machines van herstelde schijven, uw Azure-rol moet gemachtigd zijn om uit te voeren van de actie **Microsoft.KeyVault/vaults/deploy/action**. Als uw rol is niet gemachtigd dit, moet u een aangepaste rol maken met deze actie. Zie voor meer informatie, [aangepaste rollen in Azure RBAC](../role-based-access-control/custom-roles.md).
>
>

1. Query uitvoeren op de herstelde schijf-eigenschappen voor de taakdetails.

   ```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $configBlobName = $properties["Config Blob Name"]
   ```

2. Stel de context van de Azure-opslag en herstel van de JSON-configuratiebestand.

   ```powershell
   Set-AzureRmCurrentStorageAccount -Name $storageaccountname -ResourceGroupName "testvault"
   $destination_path = "C:\vmconfig.json"
   Get-AzureStorageBlobContent -Container $containerName -Blob $configBlobName -Destination $destination_path
   $obj = ((Get-Content -Path $destination_path -Raw -Encoding Unicode)).TrimEnd([char]0x00) | ConvertFrom-Json
   ```

3. De JSON-configuratiebestand gebruiken om de configuratie van de virtuele machine te maken.

   ```powershell
   $vm = New-AzureRmVMConfig -VMSize $obj.'properties.hardwareProfile'.vmSize -VMName "testrestore"
   ```

4. Koppel de besturingssysteemschijf en gegevensschijven. Deze stap bevat voorbeelden voor verschillende beheerd en versleutelde VM-configuraties. Gebruik het voorbeeld dat aansluit op uw VM-configuratie.

   * **Niet-beheerde en niet-versleutelde VM's** -gebruik het volgende voorbeeld voor niet-beheerde, niet-versleutelde VM's.

       ```powershell
       Set-AzureRmVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
       $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
       foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
       {
        $vm = Add-AzureRmVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
       }
       ```

   * **Niet-beheerde en versleutelde VM's (alleen BEK)** -voor niet-beheerde, versleutelde virtuele machines (versleuteld met behulp van de BEK alleen), moet u het geheim voor de key vault herstellen voordat u schijven kunt koppelen. Zie voor meer informatie het artikel [een versleutelde virtuele machine herstellen vanaf een herstelpunt voor Azure Backup](backup-azure-restore-key-secret.md). Het volgende voorbeeld laat zien hoe besturingssysteem en gegevensschijven koppelen voor versleutelde virtuele machines. Bij het instellen van de besturingssysteemschijf, zorg ervoor dat het relevante type besturingssysteem vermeld.

      ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $dekUrl = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      Set-AzureRmVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows/Linux
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
      foreach($dd in $obj.'properties.storageProfile'.dataDisks)
      {
       $vm = Add-AzureRmVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
      }
      ```
      Gebruik de volgende opdracht aan de versleuteling voor de gegevensschijven handmatig in te schakelen.

      ```powershell
      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $dekUrl -VolumeType Data
      ```

   * **Niet-beheerde en versleutelde VM's (BEK en KEK)** : voor niet-beheerde, versleutelde virtuele machines (versleuteld met (bek) en KEK), de sleutel en -geheim voor de key vault herstellen voordat u de schijven koppelt. Zie voor meer informatie het artikel [een versleutelde virtuele machine herstellen vanaf een herstelpunt voor Azure Backup](backup-azure-restore-key-secret.md). Het volgende voorbeeld laat zien hoe besturingssysteem en gegevensschijven koppelen voor versleutelde virtuele machines.

      ```powershell
      $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
      $kekUrl = "https://ContosoKeyVault.vault.azure.net:443/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
      $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
      Set-AzureRmVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows
      $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
      foreach($dd in $obj.'properties.storageProfile'.dataDisks)
       {
       $vm = Add-AzureRmVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
       }
      ```

      Gebruik de volgende opdracht aan de versleuteling voor de gegevensschijven handmatig in te schakelen.

      ```powershell
      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $dekUrl -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

   * **Beheerde en niet-versleutelde VM's** - voor beheerde niet-versleutelde virtuele machines en koppel de herstelde beheerde schijven. Voor gedetailleerde informatie, Zie het artikel [een gegevensschijf koppelen aan een Windows-VM met behulp van PowerShell](../virtual-machines/windows/attach-disk-ps.md).

   * **Beheerd en versleutelde virtuele machines (alleen BEK)** - voor beheerde versleutelde virtuele machines (versleuteld met behulp van de BEK alleen), koppel de herstelde beheerde schijven. Voor gedetailleerde informatie, Zie het artikel [een gegevensschijf koppelen aan een Windows-VM met behulp van PowerShell](../virtual-machines/windows/attach-disk-ps.md).
   
      Gebruik de volgende opdracht aan de versleuteling voor de gegevensschijven handmatig in te schakelen.

       ```powershell
       Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -VolumeType Data
       ```

   * **Beheerd en versleutelde virtuele machines (BEK en KEK)** - voor beheerde versleutelde virtuele machines (versleuteld met (bek) en KEK), koppel de herstelde beheerde schijven. Voor gedetailleerde informatie, Zie het artikel [een gegevensschijf koppelen aan een Windows-VM met behulp van PowerShell](../virtual-machines/windows/attach-disk-ps.md). 

      Gebruik de volgende opdracht aan de versleuteling voor de gegevensschijven handmatig in te schakelen.

      ```powershell
      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $dekUrl -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

5. Stel de netwerkinstellingen.

    ```powershell
    $nicName="p1234"
    $pip = New-AzureRmPublicIpAddress -Name $nicName -ResourceGroupName "test" -Location "WestUS" -AllocationMethod Dynamic
    $virtualNetwork = New-AzureRmVirtualNetwork -ResourceGroupName "test" -Location "WestUS" -Name "testvNET" -AddressPrefix 10.0.0.0/16
    $virtualNetwork | Set-AzureRmVirtualNetwork
    $vnet = Get-AzureRmVirtualNetwork -Name "testvNET" -ResourceGroupName "test"
    $subnetindex=0
    $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName "test" -Location "WestUS" -SubnetId $vnet.Subnets[$subnetindex].Id -PublicIpAddressId $pip.Id
    $vm=Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
    ```
6. Maak de virtuele machine.

    ```powershell  
    New-AzureRmVM -ResourceGroupName "test" -Location "WestUS" -VM $vm
    ```

## <a name="restore-files-from-an-azure-vm-backup"></a>Bestanden herstellen vanaf een back-up van virtuele Azure-machine

Naast het herstellen van schijven, kunt u ook afzonderlijke bestanden herstellen vanuit een Azure-VM back-up. De functionaliteit van de bestanden terugzetten biedt toegang tot alle bestanden in een herstelpunt. De bestanden via Verkenner beheren zoals u zou voor normale bestanden doen.

De eenvoudige stappen een bestand herstelt vanuit een Azure-VM back-up zijn:

* Selecteer de virtuele machine
* Kies een herstelpunt
* De schijven van het herstelpunt gekoppeld
* Kopieer de benodigde bestanden
* Ontkoppel de schijf


### <a name="select-the-vm"></a>Selecteer de virtuele machine

Als u de PowerShell-object waarmee de juiste back-artikel, starten van de container in de kluis en uw eigen manier omlaag in de objecthiërarchie werken. Selecteer de container die staat voor de virtuele machine, gebruikt u de **[Get-AzureRmRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupcontainer)** cmdlet en doorgeven die u wilt de **[ Get-AzureRmRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackupitem)** cmdlet.

```powershell
$namedContainer = Get-AzureRmRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM"
$backupitem = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM"
```

### <a name="choose-a-recovery-point"></a>Kies een herstelpunt

Gebruik de **[Get-AzureRmRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackuprecoverypoint)** cmdlet om alle herstelpunten voor de back-upitem weer te geven. Kies vervolgens het herstelpunt te herstellen. Als u niet zeker welk herstelpunt weet te gebruiken, is het raadzaam om te kiezen van de meest recente RecoveryPointType = AppConsistent punt in de lijst.

In het volgende script, de variabele **$rp**, is een matrix van herstelpunten voor de geselecteerde back-upitem van de afgelopen zeven dagen. De matrix is in omgekeerde volgorde gesorteerd op tijd wordt opgelost met de meest recente herstelpunt bij index 0. Gebruik de standaard PowerShell matrix indexeren om op te halen van het herstelpunt. In het voorbeeld selecteert $rp [0] het meest recente herstelpunt.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzureRmRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0]
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="mount-the-disks-of-recovery-point"></a>De schijven van het herstelpunt gekoppeld

Gebruik de **[Get-AzureRmRecoveryServicesBackupRPMountScript](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/get-azurermrecoveryservicesbackuprpmountscript)** cmdlet om op te halen van het script om alle schijven van het herstelpunt te koppelen.

> [!NOTE]
> De schijven zijn gekoppeld als iSCSI-gekoppelde schijven aan de machine waarop het script wordt uitgevoerd. Koppelen gebeurt onmiddellijk en u geen kosten.
>
>

```powershell
Get-AzureRmRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0]
```

De uitvoer lijkt op die in het volgende voorbeeld:

```
OsType  Password        Filename
------  --------        --------
Windows e3632984e51f496 V2VM_wus2_8287309959960546283_451516692429_cbd6061f7fc543c489f1974d33659fed07a6e0c2e08740.exe
```

Voer het script op de computer waar u om de bestanden te herstellen. U kunt het opgegeven wachtwoord moet invoeren voor het uitvoeren van het script. Nadat de schijven zijn gekoppeld, gebruikt u Windows Verkenner om te bladeren naar de nieuwe volumes en bestanden. Raadpleeg voor meer informatie het artikel back-up [bestanden herstellen vanuit back-up van virtuele Azure-machine](backup-azure-restore-files-from-vm.md).

### <a name="unmount-the-disks"></a>De schijven ontkoppelen

Nadat de vereiste bestanden zijn gekopieerd, gebruikt u **[uitschakelen AzureRmRecoveryServicesBackupRPMountScript](https://docs.microsoft.com/powershell/module/azurerm.recoveryservices.backup/disable-azurermrecoveryservicesbackuprpmountscript?view=azurermps-5.0.0)** naar de schijven ontkoppelen. Zorg ervoor dat de schijven ontkoppelen, zodat de toegang tot de bestanden van het herstelpunt is verwijderd.

```powershell
Disable-AzureRmRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0]
```

## <a name="next-steps"></a>Volgende stappen

Als u liever PowerShell gebruiken om u te betrekken wi th uw Azure-resources, Zie het artikel PowerShell [implementeren en beheren van back-up voor Windows Server](backup-client-automation.md). Als u DPM back-ups beheren, Zie het artikel [implementeren en beheren van Backup voor DPM](backup-dpm-automation.md). Beide van de volgende artikelen hebben een versie voor implementaties van Resource Manager en klassieke implementaties.  
