---
title: Een beschikbaarheidsset voor virtuele machines wijzigen | Microsoft Docs
description: Informatie over het wijzigen van de beschikbaarheidsset voor uw virtuele machines met Azure PowerShell en het Resource Manager-implementatiemodel.
keywords: ''
services: virtual-machines-windows
documentationcenter: ''
author: zr-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2018
ms.author: zarhoads
ms.openlocfilehash: fbbe6753bd8aa3e43c8a45db5692ce38c18de50c
ms.sourcegitcommit: 62759a225d8fe1872b60ab0441d1c7ac809f9102
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/19/2018
ms.locfileid: "49467290"
---
# <a name="change-the-availability-set-for-a-windows-vm"></a>Wijzig de beschikbaarheidsset voor een Windows-VM
De volgende stappen wordt beschreven hoe u de beschikbaarheidsset van een virtuele machine met behulp van Azure PowerShell wijzigen. Een virtuele machine kan alleen worden toegevoegd aan een beschikbaarheidsset wanneer deze wordt gemaakt. Als u wilt wijzigen van de beschikbaarheid instellen, moet u om te verwijderen en vervolgens de virtuele machine opnieuw maken. 

## <a name="change-the-availability-set"></a>De beschikbaarheidsset wijzigen 

Het volgende script geeft een voorbeeld van de vereiste gegevens te verzamelen, het verwijderen van de oorspronkelijke virtuele machine en vervolgens opnieuw worden gemaakt in een nieuwe beschikbaarheidsset.

```powershell
# Set variables
    $resourceGroup = "myResourceGroup"
    $vmName = "myVM"
    $newAvailSetName = "myAvailabilitySet"

# Get the details of the VM to be moved to the Availablity Set
    $originalVM = Get-AzureRmVM `
       -ResourceGroupName $resourceGroup `
       -Name $vmName

# Create new availability set if it does not exist
    $availSet = Get-AzureRmAvailabilitySet `
       -ResourceGroupName $resourceGroup `
       -Name $newAvailSetName `
       -ErrorAction Ignore
    if (-Not $availSet) {
    $availSet = New-AzureRmAvailabilitySet `
       -Location $originalVM.Location `
       -Name $newAvailSetName `
       -ResourceGroupName $resourceGroup `
       -PlatformFaultDomainCount 2 `
       -PlatformUpdateDomainCount 2 `
       -Sku Aligned
    }
    
# Remove the original VM
    Remove-AzureRmVM -ResourceGroupName $resourceGroup -Name $vmName    

# Create the basic configuration for the replacement VM
    $newVM = New-AzureRmVMConfig `
       -VMName $originalVM.Name `
       -VMSize $originalVM.HardwareProfile.VmSize `
       -AvailabilitySetId $availSet.Id
  
    Set-AzureRmVMOSDisk `
       -VM $newVM -CreateOption Attach `
       -ManagedDiskId $originalVM.StorageProfile.OsDisk.ManagedDisk.Id `
       -Name $originalVM.StorageProfile.OsDisk.Name `
       -Windows

# Add Data Disks
    foreach ($disk in $originalVM.StorageProfile.DataDisks) { 
    Add-AzureRmVMDataDisk -VM $newVM `
       -Name $disk.Name `
       -ManagedDiskId $disk.ManagedDisk.Id `
       -Caching $disk.Caching `
       -Lun $disk.Lun `
       -DiskSizeInGB $disk.DiskSizeGB `
       -CreateOption Attach
    }
    
# Add NIC(s)
    foreach ($nic in $originalVM.NetworkProfile.NetworkInterfaces) {
        Add-AzureRmVMNetworkInterface `
           -VM $newVM `
           -Id $nic.Id
    }

# Recreate the VM
    New-AzureRmVM `
       -ResourceGroupName $resourceGroup `
       -Location $originalVM.Location `
       -VM $newVM `
       -DisableBginfoExtension
```

## <a name="next-steps"></a>Volgende stappen

Extra opslag toevoegen aan uw virtuele machine door het toevoegen van een extra [gegevensschijf](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

