---
title: Azure DNS gebruiken voor persoonlijke domeinen | Microsoft Docs
description: Een overzicht van de persoonlijke DNS-server die als host fungeert de service op Microsoft Azure.
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2018
ms.author: victorh
ms.openlocfilehash: 2ab7070a4cf46dae543af8d3e1d688e12ec1eb2a
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173639"
---
# <a name="use-azure-dns-for-private-domains"></a>Azure DNS gebruiken voor persoonlijke domeinen
De Domain Name System of DNS is verantwoordelijk voor het omzetten van (of het oplossen van) de naam van een service aan het bijbehorende IP-adres. Een hostingservice voor DNS-domeinen, Azure DNS biedt naamomzetting met behulp van de Microsoft Azure-infrastructuur. Naast ondersteuning voor internet gerichte DNS-domeinen, ondersteunt Azure DNS nu ook persoonlijke DNS-domeinen als preview-functie. 
 
Azure DNS biedt een betrouwbare en veilige DNS-service als u wilt beheren en omzetten van domeinnamen in een virtueel netwerk zonder herconfiguratie om toe te voegen een aangepaste DNS-oplossing. Met behulp van privé-DNS-zones, kunt u uw eigen aangepaste domeinnamen in plaats van de Azure geleverde namen die momenteel beschikbaar. Met behulp van aangepaste domeinnamen, helpt u bij het aanpassen van de architectuur van uw virtuele netwerk zelf behoeften van uw organisatie. Het biedt naamomzetting voor virtuele machines (VM's) binnen een virtueel netwerk en tussen virtuele netwerken. Bovendien kunt u zonenamen configureren met een split-horizon-weergave, waarmee een persoonlijke en openbare DNS-zone voor het delen van dezelfde naam.

![DNS-overzicht](./media/private-dns-overview/scenario.png)

[!INCLUDE [private-dns-public-preview-notice](../../includes/private-dns-public-preview-notice.md)]

## <a name="benefits"></a>Voordelen

Azure DNS biedt de volgende voordelen:

* **Hiermee verwijdert u de noodzaak van aangepaste DNS-oplossingen**. Veel klanten gemaakt eerder, aangepaste DNS-oplossingen voor het beheren van DNS-zones in het virtuele netwerk. Nu kunt u DNS-zone-beheer uitvoeren met behulp van de systeemeigen Azure-infrastructuur, waardoor de werkbelasting van het maken en beheren van aangepaste DNS-oplossingen worden verwijderd.

* **Gebruik van alle algemene DNS-records typen**. Azure DNS ondersteunt A, AAAA, CNAME, MX, NS, PTR, SOA, SRV en TXT-records.

* **Automatische hostnaam recordbeheer**. Samen met uw aangepaste DNS-records hosten, bewaart Azure automatisch hostnaamrecords voor de virtuele machines in de opgegeven virtuele netwerken. In dit scenario kunt u de domeinnamen die u gebruikt zonder dat u hoeft te maken van aangepaste DNS-oplossingen of wijzig de toepassingen optimaliseren.

* **Hostnaamomzetting tussen virtuele netwerken**. In tegenstelling tot Azure verschafte hostnamen kunnen persoonlijke DNS-zones worden gedeeld tussen virtuele netwerken. Deze mogelijkheid vereenvoudigt cross-netwerk- en servicedetectie scenario's, zoals peering op virtueel netwerk.

* **Vertrouwde hulpprogramma's en gebruikerservaring**. Deze nieuwe aanbieding gebruikt, verminderen het leerproces gevestigde Azure DNS-hulpprogramma's (PowerShell, Azure resourcemanager-sjablonen en de REST-API).

* **Ondersteuning van DNS split-horizon**. U kunt zones met dezelfde naam die worden omgezet naar verschillende antwoorden uit vanuit een virtueel netwerk en het openbare internet kunt maken met Azure DNS. Een typisch scenario voor DNS-split-horizon is voor een specifieke versie van een service voor gebruik in uw virtuele netwerk.

* **Beschikbaar in alle Azure-regio's**. De functie voor Azure DNS private zones is beschikbaar in alle Azure-regio's in de openbare cloud van Azure. 


## <a name="capabilities"></a>Functionaliteit

Azure DNS biedt de volgende mogelijkheden:
 
* **Automatische registratie van virtuele machines van één virtueel netwerk dat gekoppeld aan een privézone als een registration virtual network**. De virtuele machines zijn ingeschreven (toegevoegd) aan de persoonlijke zone als A-records die verwijst naar hun privé-IP's. Als een virtuele machine in de registratie van een virtueel netwerk wordt verwijderd, wordt Azure de bijbehorende DNS-server ook automatisch verwijderd van de gekoppelde privézone. 

  > [!NOTE]
  > Standaard worden virtuele registratienetwerken ook fungeren als het probleem zou moeten virtuele netwerken in de zin dat DNS-omzetting op basis van de zone in elk van de virtuele machines binnen het registration virtual network werkt. 

* **Forward DNS-omzetting wordt ondersteund tussen virtuele netwerken die zijn gekoppeld aan de privézone als het probleem zou moeten virtuele netwerken**. Voor cross-virtueel netwerk DNS-omzetting is het geen expliciete afhankelijkheid zodanig dat de virtuele netwerken met elkaar zijn gekoppeld. Klanten kunnen echter ervoor virtuele netwerken voor andere scenario's (bijvoorbeeld HTTP-verkeer).

* **Omgekeerde DNS-zoekopdracht wordt ondersteund binnen het bereik van het virtuele netwerk**. Omgekeerde DNS-zoekactie voor een privé IP-adres binnen het virtuele netwerk dat is toegewezen aan een privézone wordt de FQDN-naam met de naam van de host/record, evenals de naam van de zone als het achtervoegsel geretourneerd. 


## <a name="limitations"></a>Beperkingen

Azure DNS is onderworpen aan de volgende beperkingen:

* Registratie van slechts één virtueel netwerk is toegestaan per privézone.
* Virtuele netwerken mogen maximaal 10 oplossingen per privézone.
* Een specifiek virtueel netwerk kan worden gekoppeld aan de zone met slechts één persoonlijke als een registration virtual network.
* Een specifiek virtueel netwerk kan maximaal 10 private zones als een virtueel netwerk van het probleem zou moeten worden gekoppeld.
* Als een virtueel netwerk van de registratie is opgegeven, de DNS-records voor de virtuele machines van dit virtuele netwerk die zijn geregistreerd bij de privézone zijn niet kan worden weergegeven of worden opgehaald uit de Azure Powershell en Azure CLI-API's, maar de records van de virtuele machine inderdaad worden geregistreerd en wordt is opgelost.
* Omgekeerde DNS-geldt alleen voor privé-IP-adresruimte in het registration virtual network.
* Reverse-DNS voor een privé IP-adres dat niet is geregistreerd in het privé-zone (bijvoorbeeld een privé IP-adres voor een virtuele machine in een virtueel netwerk dat is gekoppeld als een resolutie van virtueel netwerk met een particulier zone) retourneert *internal.cloudapp.net* Als de DNS-achtervoegsel. Dit achtervoegsel is echter niet omgezet. 
* Het virtuele netwerk moet leeg zijn (dat wil zeggen, er zijn geen records VM bestaat) wanneer er in eerste instantie (dat wil zeggen, voor de eerste keer) koppelingen naar een privézone als een registratie- of resolutienetwerken virtueel netwerk. Het virtuele netwerk kan echter vervolgens worden niet-lege voor toekomstige koppelen als een virtueel netwerk van registratie- of resolutienetwerken naar andere private zones. 
* Voorwaardelijk doorsturen wordt op dit moment niet ondersteund (bijvoorbeeld voor het inschakelen van het probleem zou moeten tussen Azure en on-premises netwerken). Zie voor meer informatie over hoe klanten in dit scenario via andere methoden kunnen realiseren [naamomzetting voor VM's en rolexemplaren](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

Voor veelgestelde vragen en antwoorden over privézones in Azure DNS, met inbegrip van specifieke DNS-registratie en de resolutie gedrag kunt u verwachten voor bepaalde soorten bewerkingen, Zie [Veelgestelde vragen over](./dns-faq.md#private-dns).  


## <a name="pricing"></a>Prijzen

De functie Privé DNS-zones is gratis tijdens de openbare preview. Bij algemene beschikbaarheid biedt de functie een op gebruik gebaseerde prijsmodel vergelijkbaar met dat van de bestaande Azure DNS biedt. 


## <a name="next-steps"></a>Volgende stappen

Informatie over het maken van een privé-zone in Azure DNS met behulp van [Azure PowerShell](./private-dns-getstarted-powershell.md) of [Azure CLI](./private-dns-getstarted-cli.md).

Meer informatie over sommige algemene [privézone-scenario's](./private-dns-scenarios.md) die u kunt realiseren met privézones in Azure DNS.

Voor algemene vragen en antwoorden over privézones in Azure DNS, met inbegrip van specifiek gedrag kunt u verwachten voor bepaalde soorten bewerkingen, Zie [Veelgestelde vragen over](./dns-faq.md#private-dns). 

Meer informatie over DNS-zones en records, recentst [DNS-zones en records overzicht](dns-zones-records.md).

Informatie over enkele van de andere belangrijke [netwerkmogelijkheden](../networking/networking-overview.md) van Azure. 

