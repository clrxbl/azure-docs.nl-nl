---
title: 'Azure Active Directory Domain Services: Functies | Microsoft Docs'
description: Functies van Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 8d1c3eb3-1022-4add-a919-c98cc6584af1
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: maheshu
ms.openlocfilehash: a64401792fd034fde98fe1330340b1ffaa7dfcc1
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505409"
---
# <a name="azure-ad-domain-services"></a>Azure AD Domain Services
## <a name="features"></a>Functies
De volgende functies zijn beschikbaar in Azure AD Domain Services beheerde domein.

* **Eenvoudige implementatie-ervaring:** u Azure AD Domain Services voor uw Azure AD-directory met behulp van slechts een paar klikken kunt inschakelen. Uw beheerde domein bevat alleen in de cloud gebruikersaccounts en gebruikersaccounts die zijn gesynchroniseerd vanuit een on-premises directory.
* **Ondersteuning voor het lid van domein:** kunt u eenvoudig domein computers in uw beheerde domein is beschikbaar in Azure virtual network. De domein-ervaring op Windows-client en Server-besturingssystemen werkt naadloos met domeinen die zijn verwerkt door Azure AD Domain Services. Ook kunt u geautomatiseerde domeindeelname hulpprogramma's op basis van deze domeinen.
* **Één domein exemplaar per Azure AD-directory:** kunt u één Active Directory-domein voor elke Azure AD-directory.
* **Domeinen met aangepaste namen maken:** u domeinen met aangepaste namen (bijvoorbeeld ' contoso100.com') met behulp van Azure AD Domain Services kunt maken. U kunt een van beide namen geverifieerde of niet-geverifieerd domein gebruiken. (Optioneel) u kunt ook een domein maken met het ingebouwde domeinachtervoegsel (dat wil zeggen, ' *. onmicrosoft.com') die worden aangeboden door uw Azure AD-directory.
* **Geïntegreerd met Azure AD:** u hoeft niet te configureren of beheren van replicatie naar Azure AD Domain Services. Gebruikersaccounts, groepslidmaatschappen en referenties van gebruiker (wachtwoorden) van uw Azure AD-directory zijn automatisch beschikbaar in Azure AD Domain Services. Nieuwe gebruikers, groepen of wijzigingen in kenmerken van uw Azure AD-tenant of uw on-premises directory worden automatisch gesynchroniseerd met Azure AD Domain Services.
* **NTLM en Kerberos-verificatie:** met ondersteuning voor NTLM en Kerberos-verificatie, kunt u toepassingen die afhankelijk van Windows-Integrated verificatie zijn implementeren.
* **Gebruik uw zakelijke referenties/wachtwoorden:** wachtwoorden voor gebruikers in uw Azure AD-tenant in combinatie met Azure AD Domain Services. Gebruikers kunnen hun bedrijfsreferenties voor toegang tot de domein-machines gebruiken, meld u interactief of via Extern bureaublad en verificatie op basis van het beheerde domein.
* **LDAP-binding & LDAP lezen ondersteuning:** kunt u toepassingen die afhankelijk van LDAP-bindingen zijn voor verificatie van gebruikers in domeinen die zijn verwerkt door Azure AD Domain Services. Toepassingen die gebruikmaken van LDAP-leesbewerkingen in query gebruiker/computer kenmerken uit de map kunnen bovendien ook worden gebruikt op basis van Azure AD Domain Services.
* **Secure LDAP (LDAPS):** kunt u toegang tot de map via veilige LDAP (LDAPS) inschakelen. Toegang van Secure LDAP is beschikbaar in het virtuele netwerk standaard. U kunt echter ook optioneel toegang van secure LDAP inschakelen via het internet.
* **Group Policy:** kunt u één ingebouwde groepsbeleidsobject elke voor de gebruikers en computers containers om af te dwingen van naleving van beveiligingsbeleid voor accounts van gebruikers en computers domein vereist. U kunt ook uw eigen aangepaste GPO's maken en deze toewijzen aan aangepaste organisatie-eenheden te [beheren van Groepsbeleid](active-directory-ds-admin-guide-administer-group-policy.md).
* **DNS beheren:** leden van de groep 'AAD DC Administrators' kunnen DNS beheren voor uw beheerde domein met bekende DNS-beheer-hulpprogramma's zoals de DNS-beheer van MMC-module.
* **Aangepaste organisatie-eenheden (OU's) maken:** leden van de groep 'AAD DC Administrators' aangepaste organisatie-eenheden kunnen maken in het beheerde domein. Deze gebruikers krijgen volledige beheerdersrechten over aangepaste organisatie-eenheden, zodat u ze kunnen toevoegen/verwijderen serviceaccounts, computers, groepen enzovoort binnen deze aangepaste organisatie-eenheden.
* **In veel Azure wereldwijde regio's beschikbaar:** Zie de [Azure-services per regio](https://azure.microsoft.com/regions/#services/) pagina om te bekijken van de Azure-regio's waar Azure AD Domain Services beschikbaar is.
* **Hoge beschikbaarheid:** Azure AD Domain Services biedt hoge beschikbaarheid voor uw domein. Deze functie biedt de garantie van hogere service uptime en herstelmogelijkheden bij fouten. Ingebouwde voor health monitoring biedt automatisch herstel van fouten door nieuwe instanties om mislukte exemplaren vervangen en om doorlopende service voor uw domein.
* **Beveiliging met accountvergrendeling AD:** gebruikersaccounts zijn vergrendeld voor 30 minuten als vijf ongeldige wachtwoorden worden gebruikt binnen twee minuten. Accounts zijn automatisch ontgrendelen na 30 minuten.
* **Beheerprogramma's bekend:** u vertrouwde Windows Server Active Directory-beheerprogramma's zoals de Active Directory-beheercentrum of Active Directory PowerShell kunt gebruiken voor het beheer van beheerde domeinen.
