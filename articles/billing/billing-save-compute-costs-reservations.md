---
title: Wat zijn Azure-reserveringen? | Microsoft Docs
description: Meer informatie over Azure-reserveringen en prijzen op te slaan op uw virtuele machines, SQL-databases, Azure Cosmos DB en andere resourcekosten.
services: billing
documentationcenter: ''
author: yashesvi
manager: yashesvi
editor: ''
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2018
ms.author: cwatson
ms.openlocfilehash: a0fb5eccf14aee07005ab345f1af293f341f9215
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/28/2018
ms.locfileid: "47422946"
---
# <a name="what-are-azure-reservations"></a>Wat zijn Azure-reserveringen?

Azure-reserveringen helpt u geld besparen door vooraf betalen voor een eenjarige of driejarige van de virtuele machine, SQL Database-rekencapaciteit, doorvoer van Azure Cosmos DB of andere Azure-resources. Vooraf betaalt, kunt u krijg een korting op de resources die u gebruikt. Uw virtuele machine, compute-SQL-database, Azure Cosmos DB kunnen aanzienlijk verkorten door reserveringen of andere bron kosten tot 72% bij betalen per gebruik. Reserveringen bieden een korting van facturering en hebben geen invloed op de runtimestatus van uw resources.

U kunt een reservering kopen de [Azure-portal](https://aka.ms/reservations). Zie de volgende onderwerpen voor meer informatie:

- [Vooruitbetalen voor virtuele Machines met Azure gereserveerde VM-instanties](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Betaal vooruit voor SQL Database-compute-resources met Azure SQL Database gereserveerde capaciteit](../sql-database/sql-database-reserved-capacity.md)
- [Betaal vooruit voor Azure Cosmos DB-resources met Azure Cosmos DB gereserveerde capaciteit](../cosmos-db/cosmos-db-reserved-capacity.md)

## <a name="why-should-i-buy-a-reservation"></a>Waarom moet ik een reservering kopen?

Hebt u virtuele machines, Azure Cosmos DB of SQL-databases die langdurig worden uitgevoerd, biedt een reservering kopen u de meest voordelige optie zijn. Bijvoorbeeld, als u vier exemplaren van een service zonder een reservering continu uitvoeren, in rekening gebracht tegen betalen per gebruik-tarieven. Als u een reservering voor deze resources koopt, krijgt u onmiddellijk de reserveringskorting. De resources zijn niet meer in rekening gebracht tegen de betalen naar gebruik-tarieven.

## <a name="what-charges-does-a-reservation-cover"></a>Wat in rekening gebracht, wordt een reservering cover?

- Gereserveerde VM-instantie: Een reservering heeft alleen betrekking op de kosten voor rekenuren van de virtuele machine. Het bedekt geen extra kosten voor software, netwerken en opslag.
- SQL Database vCore gereserveerd: alleen de berekeningskosten opgenomen in een reservering zijn. De licentie wordt apart in rekening gebracht.
- Azure Cosmos DB gereserveerde capaciteit: een reservering geldt voor ingerichte doorvoer voor uw resources, het heeft geen betrekking op de kosten voor opslag en netwerken. 

Voor virtuele machines van Windows en SQL Database, kunt u de licentiekosten met dekt [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).

## <a name="whos-eligible-to-purchase-a-reservation"></a>Wie komt in aanmerking voor een reservering kopen?

Deze abonnementen typen Azure-klanten kunnen een reservering kopen:

- Abonnementtype Enterprise-overeenkomst (MS-AZR - 0017P).
- Abonnementtype [Betalen per gebruik](https://azure.microsoft.com/offers/ms-azr-0003p/) (MS-AZR-003P). U moet de 'Eigenaar'-rol hebben op het abonnement op een reservering kopen.
- Cloud Solution Provider (CSP) partners kunnen Azure-portal gebruiken of [Partner Center](https://docs.microsoft.com/partner-center/azure-reservations) Azure reserveringen kopen.

Een reserveringskorting is alleen van toepassing op resources die zijn gekoppeld aan Enterprise, betalen per gebruik of CSP-abonnementstypen.

## <a name="how-is-a-reservation-billed"></a>Hoe wordt een reservering gefactureerd?

De reservering wordt verrekend met de betalingswijze die is gekoppeld aan het abonnement. Als u een Enterprise-abonnement hebt, worden de reserveringskosten voor wordt afgetrokken van uw monetaire toezeggingsbedrag. Als uw monetaire toezeggingsbedrag dekt de kosten van de reservering niet, u wordt gefactureerd voor de overschrijding. Als u een betalen per gebruik-abonnement hebt, wordt de creditcard die u voor uw account hebt wordt onmiddellijk gefactureerd. Als u wordt gefactureerd per factuur, ziet u de kosten in rekening gebracht op uw volgende factuur.

## <a name="how-is-the-reservation-discount-applied"></a>Hoe wordt de reserveringskorting toegepast?

De reserveringskorting is van toepassing op het gebruik van bronnen die overeenkomt met de kenmerken die u selecteert wanneer u de reservering kopen. De kenmerken omvatten het bereik waarin de overeenkomende virtuele machines, SQL databases, Azure Cosmos DB of andere bronnen wordt uitgevoerd. Bijvoorbeeld, als u een reserveringskorting voor vier standaard D2 virtuele machines in de regio VS-West wilt, selecteer het abonnement waarin de virtuele machines worden uitgevoerd. Als de virtuele machines worden uitgevoerd in verschillende abonnementen binnen uw inschrijvingsaccount, selecteert u het bereik als een gedeeld. Gedeeld bereik kunt de reserveringskorting moet worden toegepast voor abonnementen. U kunt het bereik wijzigen nadat u een reservering kopen. Zie voor meer informatie, [Azure-reserveringen beheren](billing-manage-reserved-vm-instance.md).

Een reserveringskorting is alleen van toepassing op resources die zijn gekoppeld aan Enterprise, betalen per gebruik of CSP-abonnementstypen. Resources die worden uitgevoerd in een abonnement met andere typen ontvangen niet de reserveringskorting. Enterprise Dev/Test-abonnementen zijn niet voor enterprise-inschrijvingen, in aanmerking komen voor de voordelen van de reservering.

Zie voor meer informatie over de invloed van reserveringen op uw facturering, de volgende onderwerpen:

-  [Korting op Azure Reserved VM Instances begrijpen](billing-understand-vm-reservation-charges.md)
- [Azure reserveringskorting begrijpen](billing-understand-vm-reservation-charges.md)
- [Azure Cosmos DB-reserveringskorting begrijpen](billing-understand-cosmosdb-reservation-charges.md)
- [Meer informatie over Azure reserveringskorting en het gebruik voor SUSE](billing-understand-suse-reservation-charges.md)

## <a name="what-happens-when-the-reservation-term-expires"></a>Wat gebeurt er wanneer de reserveringstermijn is verlopen?

De facturering korting verloopt aan het einde van de reserveringstermijn, en de virtuele machine, SQL database, Azure Cosmos DB of andere resource wordt gefactureerd voor de betaalt u Ga-prijs. Azure-reserveringen verlengen niet automatisch. Als u wilt doorgaan met het ophalen van de korting voor facturering, moet u een nieuwe reservering voor in aanmerking komende services en software kopen.

## <a name="discount-applies-to-different-sizes-with-instance-size-flexibility"></a>Korting geldt voor verschillende grootten met een grootte-instantieflexibiliteit

Wanneer u een reservering koopt, wordt de korting kunt toepassen op andere exemplaren met kenmerken die zich binnen de groep met dezelfde grootte. De flexibiliteit van de dekking van de korting is afhankelijk van het type reservering en de kenmerken die u kiest wanneer u de reservering kopen.

- Gereserveerde VM-instanties: Wanneer u koopt de reservering, als u selecteert **geoptimaliseerd voor**: **exemplaar grootte flexibiliteit**, de dekking van de korting is afhankelijk van de VM-grootte die u selecteert. De reservering kunt toepassen op de grootten van virtuele machines (VM's) in de groep voor reeks van dezelfde grootte. Zie voor meer informatie, [VM-grootte flexibiliteit in combinatie met gereserveerde VM-instanties](../virtual-machines/windows/reserved-vm-instance-size-flexibility.md).
- Plan voor SUSE Linux Enterprise software: de dekking van de korting is afhankelijk van de vcpu's van de virtuele machines waarop u de SUSE-software uitvoert. Zie voor meer informatie, [te begrijpen hoe de korting voor SUSE Linux Enterprise software plan wordt toegepast](billing-understand-suse-reservation-charges.md).
- SQL-Database gereserveerde capaciteit: de dekking van de korting is afhankelijk van de prestatielaag die u kiest. Zie voor meer informatie, [te begrijpen hoe een Azure reserveringskorting wordt toegepast](billing-understand-reservation-charges.md).
- Azure Cosmos DB gereserveerde capaciteit: de dekking van de korting is afhankelijk van de ingerichte doorvoer. Zie voor meer informatie, [te begrijpen hoe een Azure Cosmos DB-reserveringskorting wordt toegepast](billing-understand-cosmosdb-reservation-charges.md).

## <a name="next-steps"></a>Volgende stappen

Start op uw virtuele machines opslaan door aan te schaffen een [gereserveerde VM-instantie](../virtual-machines/windows/prepay-reserved-vm-instances.md), [SQL-Database gereserveerde capaciteit](../sql-database/sql-database-reserved-capacity.md), of [Azure Cosmos DB gereserveerde capaciteit](../cosmos-db/cosmos-db-reserved-capacity.md).

Zie voor meer informatie over Azure-reserveringen, de volgende artikelen:

- [Azure-reserveringen beheren](billing-manage-reserved-vm-instance.md)
- [Gebruik van de reservering voor uw abonnement op gebruiksbasis begrijpen](billing-understand-reserved-instance-usage.md)
- [Inzicht in gebruik van de reservering voor uw Enterprise-inschrijving](billing-understand-reserved-instance-usage-ea.md)
- [Kosten van de Windows-software is niet opgenomen in de reserveringen](billing-reserved-instance-windows-software-costs.md)
- [Azure-reserveringen in programma Partner Center Cloud Solution Provider (CSP)](https://docs.microsoft.com/partner-center/azure-reservations)

## <a name="need-help-contact-support"></a>Hulp nodig? Contact opnemen met ondersteuning

Als u nog meer vragen hebt, [contact op met ondersteuning](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) om uw probleem snel worden opgelost.
