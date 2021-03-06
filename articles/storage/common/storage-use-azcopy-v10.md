---
title: Gegevens kopiëren of verplaatsen naar Azure Storage met AzCopy v10 (Preview) | Microsoft Docs
description: De v10 AzCopy gebruiken (Preview)-hulpprogramma voor het verplaatsen of kopiëren van gegevens naar of van blob-, tabel en bestandsinhoud. Gegevens kopiëren naar Azure Storage van lokale bestanden en gegevens binnen of tussen opslagaccounts kopiëren. Migreer uw gegevens eenvoudig naar Azure Storage.
services: storage
author: artemuwka
ms.service: storage
ms.topic: article
ms.date: 10/09/2018
ms.author: artemuwka
ms.component: common
ms.openlocfilehash: a1b183e5b0929a2149502aa340e2e69c725dba6d
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/12/2018
ms.locfileid: "49168260"
---
# <a name="transfer-data-with-the-azcopy-v10-preview"></a>Gegevensoverdracht met het AzCopy-v10 (Preview)

AzCopy v10 (Preview) is de generatie-opdrachtregelprogramma voor het kopiëren van gegevens naar/van Microsoft Azure BLOB Storage en File storage, een opdrachtregelinterface opnieuw ontworpen en een nieuwe architectuur voor krachtige, betrouwbare de gegevensoverdracht biedt. Met behulp van AzCopy kunt u gegevens tussen een bestandssysteem en een opslagaccount of tussen opslagaccounts kopiëren.

## <a name="whats-new-in-azcopy-v10"></a>Wat is er nieuw in AzCopy v10

- Synchroniseren van een bestandssysteem naar Azure Blob of vice versa. Gebruik `azcopy sync <source> <destination>`. Ideaal voor scenario's voor incrementele kopie.
- Biedt ondersteuning voor Azure Data Lake Storage Gen2 API's. Gebruik `myaccount.dfs.core.windows.net` als een URI naar de ADLS Gen2 API's aanroepen.
- Ondersteunt het kopiëren van een hele account (alleen voor Blob-service) naar een ander account.
- Account kopiëren-account maakt nu gebruik van de nieuwe [plaatsen van URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API's. Er is geen gegevensoverdracht naar de client is vereist waardoor de overdracht sneller!
- Lijst/verwijderen bestanden en blobs in een opgegeven pad.
- Ondersteunt jokertekenpatronen, evenals--opnemen en--uitsluiten van vlaggen in een pad.
- Verbeterde tolerantie: elk exemplaar AzCopy maakt u de volgorde van een taak en een bijbehorende logboekbestand. U kunt bekijken en vorige taken opnieuw starten en mislukte taken hervatten. AzCopy wordt ook automatisch opnieuw geprobeerd een overdracht na een storing.
- Algemene verbeteringen.

## <a name="download-and-install-azcopy"></a>Download en installeer AzCopy

### <a name="latest-preview-version-v10"></a>Meest recente preview-versie (v10)

De meest recente preview-versie van AzCopy downloaden:
- [Windows](https://aka.ms/downloadazcopy-v10-windows)
- [Linux](https://aka.ms/downloadazcopy-v10-linux)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac)

### <a name="latest-production-version-v81"></a>Nieuwste productieversie (v8.1)

Download de [nieuwste productieversie van AzCopy voor Windows](https://aka.ms/downloadazcopy).

### <a name="azcopy-supporting-table-storage-service-v73"></a>AzCopy Table storage-service (v7.3) ondersteunen

Download de [AzCopy v7.3 kopiëren van gegevens naar/van Microsoft Azure Table storage-service ondersteunt](https://aka.ms/downloadazcopynet).

## <a name="post-installation-steps"></a>Stappen na de installatie

AzCopy v10 is niet vereist voor een installatie. Open een voorkeurs-opdrachtregelprogramma en navigeer naar de map waar de `azcopy.exe` uitvoerbare bestand zich bevindt. Indien gewenst, kunt u de locatie van de AzCopy-map toevoegen aan het systeempad staat.

## <a name="authentication-options"></a>Verificatie-opties

AzCopy v10 kunt u de volgende opties gebruiken voor verificatie met Azure Storage:
- Azure Active Directory. Gebruik ```.\azcopy login``` zich aanmelden met Azure Active Directory.  De gebruiker moet beschikken over ['Blob Gegevensbijdrager voor' rol](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac) te schrijven naar Blob-opslag met Azure Active Directory-verificatie.
- SAS-token dat moet worden toegevoegd aan het pad van de Blob. U kunt met behulp van Azure Portal, SAS-token genereren [Opslagverkenner](https://blogs.msdn.microsoft.com/jpsanders/2017/10/12/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer/), [PowerShell](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestorageblobsastoken?view=azurermps-6.9.0), of andere hulpprogramma's van uw keuze. Zie voor meer informatie, [voorbeelden](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2).

## <a name="getting-started"></a>Aan de slag

AzCopy v10 is een eenvoudige syntaxis zelf beschreven. De algemene syntaxis ziet er als volgt uit:

```azcopy
.\azcopy <command> <arguments> --<flag-name>=<flag-value>
# Example:
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/containersastoken" --recursive=true
```

Hier volgt hoe krijgt u een lijst van beschikbare opdrachten:

```azcopy
.\azcopy -help
# Using the alias instead
.\azcopy -h
```

Om te zien in de help-pagina en voorbeelden voor een specifieke opdracht de onderstaande opdracht uitvoeren:

```azcopy
.\azcopy <cmd> -help
# Example:
.\azcopy cp -h
```

## <a name="copy-data-to-azure-storage"></a>Gegevens kopiëren naar Azure Storage

Gebruik de opdracht kopiëren om over te dragen gegevens van de bron naar de bestemming. De bron-/ doel mag a:
- Lokaal bestandssysteem
- Azure-Blob/virtuele Directory/Container URI
- Azure-bestand/map/bestandsshare URI
- Azure Data Lake Storage Gen2 bestandssysteem/map/bestand URI

> [!NOTE]
> Op dit moment ondersteunt AzCopy v10 alleen blok-blobs tussen twee opslagaccounts kopiëren.

```azcopy
.\azcopy copy <source path> <destination path> --<flag-name>=<flag-value>
# Using alias instead
.\azcopy cp <source path> <destination path> --<flag-name>=<flag-value>
```

De volgende opdracht worden alle bestanden onder de map C:\local\path recursief geüpload naar de container 'mycontainer1':

```azcopy
.\azcopy cp "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true
```

De volgende opdracht worden alle bestanden onder de map C:\local\path (zonder recursing in de submappen) geüpload naar de container 'mycontainer1':

```azcopy
.\azcopy cp "C:\local\path\*" "https://account.blob.core.windows.net/mycontainer1<sastoken>"
```

Als u meer voorbeelden, gebruikt u de volgende opdracht uit:

```azcopy
.\azcopy cp -h
```

## <a name="copy-data-between-two-storage-accounts"></a>Kopiëren van gegevens tussen twee opslagaccounts

Maakt gebruik van kopiëren van gegevens tussen twee opslagaccounts de [blok plaatsen van URL](https://docs.microsoft.com/rest/api/storageservices/put-block-from-url) API en geen gebruik van netwerkbandbreedte van de client-computer. Gegevens worden gekopieerd tussen twee Azure Storage-servers rechtstreeks terwijl AzCopy gewoon de kopieerbewerking wordt georganiseerd. 

Als u wilt kopiëren van gegevens tussen twee opslagaccounts, gebruik de volgende opdracht:
```azcopy
.\azcopy cp "https://myaccount.blob.core.windows.net/<sastoken>" "https://myotheraccount.blob.core.windows.net/<sastoken>" --recursive=true
```

> [!NOTE]
> De opdracht wordt het inventariseren van alle blob-containers en kopieer deze naar de doelaccount. Op dit moment ondersteunt AzCopy v10 alleen blok-blobs tussen twee opslagaccounts kopiëren. Alle andere storage-account-objecten (toevoeg-blobs, pagina-blobs, bestanden, tabellen en wachtrijen) wordt overgeslagen.

## <a name="copy-a-vhd-image-to-a-storage-account"></a>Een VHD-installatiekopie kopiëren naar een opslagaccount

AzCopy v10 standaard worden gegevens geüpload naar blok-blobs. Echter, als een bronbestand vhd-extensie heeft, AzCopy v10 wordt standaard uploaden naar een pagina-blob. Dit gedrag is niet geconfigureerd.

## <a name="sync-incremental-copy-and-delete"></a>Synchronisatie: de incrementele kopie en verwijderen

> [!NOTE]
> Synchronisatieopdracht synchroniseert de inhoud van bron naar doel en dit omvat het verwijderen van de doel-bestanden als die nog niet bestaan in de bron. Zorg ervoor dat u de bestemming die u van plan bent om te synchroniseren.

Als u wilt synchroniseren van uw lokale systeem naar een opslagaccount, gebruik de volgende opdracht:

```azcopy
.\azcopy sync "C:\local\path" "https://account.blob.core.windows.net/mycontainer1<sastoken>" --recursive=true
```

Op dezelfde manier kunt u een Blob-container naar een lokaal bestandssysteem synchroniseren:

```azcopy
# If you're using Azure Active Directory authentication the sastoken is not required
.\azcopy sync "https://account.blob.core.windows.net/mycontainer1" "C:\local\path" --recursive=true
```

De opdracht kunt u de bron naar de bestemming op basis van laatst gewijzigde tijdstempels incrementeel te synchroniseren. Als u toevoegen of verwijderen van een bestand in de bron, doet AzCopy v10 hetzelfde als in de bestemming.

## <a name="advanced-configuration"></a>Geavanceerde configuratie

### <a name="configure-proxy-settings"></a>Proxy-instellingen configureren

Voor het configureren van de proxy-instellingen voor AzCopy v10, stel de omgeving variabele https_proxy met de volgende opdracht:

```cmd
# For Windows:
set https_proxy=<proxy IP>:<proxy port>
# For Linux:
export https_proxy=<proxy IP>:<proxy port>
# For MacOS
export https_proxy=<proxy IP>:<proxy port>
```

### <a name="optimize-throughput"></a>Optimaliseren van doorvoer

Stel de omgevingsvariabele AZCOPY_CONCURRENCY_VALUE voor het configureren van het aantal gelijktijdige aanvragen en beheren van de prestaties van de netwerkdoorvoer en resourceverbruik. De waarde is standaard ingesteld op 300. Vermindering van de waarde, wordt de bandbreedte en CPU-gebruik door AzCopy v10 beperkt.

```cmd
# For Windows:
set AZCOPY_CONCURRENCY_VALUE=<value>
# For Linux:
export AZCOPY_CONCURRENCY_VALUE=<value>
# For MacOS
export AZCOPY_CONCURRENCY_VALUE=<value>
```

## <a name="troubleshooting"></a>Problemen oplossen

AzCopy v10 logboekbestanden en plan bestanden voor alle taken gemaakt. U kunt de logboeken kunt gebruiken om te onderzoeken en oplossen van mogelijke problemen. De logboeken van de status van de fout (UPLOADFAILED, COPYFAILED en DOWNLOADFAILED), bevat het volledige pad en de reden van de fout. De taaklogboeken van de en plan-bestanden bevinden zich in de map % USERPROFILE\\.azcopy map.

### <a name="review-the-logs-for-errors"></a>Bekijk de logboeken voor fouten

De volgende opdracht ontvangt alle fouten met UPLOADFAILED status van het logboek 04dc9ca9-158f-7945-5933-564021086c79:

```azcopy
cat 04dc9ca9-158f-7945-5933-564021086c79.log | grep -i UPLOADFAILED
```

### <a name="view-and-resume-jobs"></a>Weergeven en taken hervatten

Elke overdrachtbewerking wordt een AzCopy-taak maken. Hier vindt u de geschiedenis van taken met behulp van de volgende opdracht uit:

```azcopy
.\azcopy jobs list
```

Als u wilt weergeven van statistieken voor de taak, gebruik de volgende opdracht:

```azcopy
.\azcopy jobs show <job-id>
```

Als u wilt filteren de overdrachten op status, gebruik de volgende opdracht:

```azcopy
.\azcopy jobs show <job-id> --with-status=Failed
```

U kunt een taak is mislukt/geannuleerd met de id samen met het SAS-token (dit is niet persistent uit veiligheidsoverwegingen) hervatten:

```azcopy
.\azcopy jobs resume <jobid> --sourcesastokenhere --destinationsastokenhere
```

## <a name="next-steps"></a>Volgende stappen

Uw feedback wordt altijd verwelkomde. Als u vragen hebt, problemen of algemene feedback over deze indienen https://github.com/Azure/azure-storage-azcopy. Bedankt.