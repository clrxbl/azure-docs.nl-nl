---
title: Azure Stack-datacenter-integratie - eindpunten publiceren | Microsoft Docs
description: Informatie over het publiceren van Azure Stack-eindpunten in uw datacenter
services: azure-stack
author: jeffgilb
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 09/13/2018
ms.author: jeffgilb
ms.reviewer: wamota
keywords: ''
ms.openlocfilehash: bc07d0e7aff2bf9263d89dffccc47ba806ff0fd1
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804205"
---
# <a name="azure-stack-datacenter-integration---publish-endpoints"></a>Azure Stack-datacenter-integratie - eindpunten publiceren

Azure Stack, stelt u virtuele IP-adressen (VIP's) voor de functies van de infrastructuur. Deze VIP's worden toegewezen uit het openbare IP-adresgroep. Elke VIP is beveiligd met een toegangsbeheerlijst (ACL) in de software-defined network-laag. ACL's worden ook gebruikt voor de fysieke switches (tors filteren en BMC) voor het verder beperken van de oplossing. Een DNS-vermelding wordt voor elk eindpunt in de externe DNS-zone die tijdens de implementatie opgegeven gemaakt.


De volgende Architectuurdiagram ziet u de verschillende lagen en ACL's:

![Structurele afbeelding](media/azure-stack-integrate-endpoints/Integrate-Endpoints-01.png)

## <a name="ports-and-protocols-inbound"></a>Poorten en protocollen (inkomend)

Een set met infrastructuur voor VIP's is vereist voor publicatie Azure Stack-eindpunten met externe netwerken. De *eindpunt (VIP)* tabel ziet u elk eindpunt, de vereiste poort en protocol. Zie de Implementatiedocumentatie voor specifieke resource provider voor eindpunten die extra bronnen-providers, zoals de SQL-resourceprovider.

Interne infrastructuur voor VIP's worden niet weergegeven omdat ze niet vereist voor publiceren op Azure Stack.

> [!Note]  
> Gebruiker VIP's zijn dynamisch, gedefinieerd door de gebruikers zelf geen controle door de Azure Stack-operator.

|Eindpunt (VIP)|DNS-host A-record|Protocol|Poorten|
|---------|---------|---------|---------|
|AD FS|Adfs.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Portal (beheerder)|Adminportal.*&lt;region>.&lt;fqdn>*|HTTPS|443<br>12495<br>12499<br>12646<br>12647<br>12648<br>12649<br>12650<br>13001<br>13003<br>13010<br>13011<br>13012<br>13020<br>13021<br>13026<br>30015|
|Adminhosting | *.adminhosting. \<regio >. \<FQDN-naam > | HTTPS | 443 |
|Azure Resource Manager (beheerder)|Adminmanagement.*&lt;region>.&lt;fqdn>*|HTTPS|443<br>30024|
|Portal (gebruiker)|Portal.*&lt;region>.&lt;fqdn>*|HTTPS|443<br>12495<br>12649<br>13001<br>13010<br>13011<br>13012<br>13020<br>13021<br>30015<br>13003|
|Azure Resource Manager (gebruiker)|Management.*&lt;region>.&lt;fqdn>*|HTTPS|443<br>30024|
|Graph|Graph.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Certificaatintrekkingslijst|Crl.*&lt;region>.&lt;fqdn>*|HTTP|80|
|DNS|&#42;.*&lt;region>.&lt;fqdn>*|TCP EN UDP|53|
|Die als host fungeert | * .hosting. \<regio >. \<FQDN-naam > | HTTPS | 443 |
|Key Vault (gebruiker)|&#42;.vault.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Key Vault (beheerder)|&#42;.adminvault.*&lt;region>.&lt;fqdn>*|HTTPS|443|
|Opslagwachtrij|&#42;.queue.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Storage-tabel|&#42;.table.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|Storage Blob|&#42;.blob.*&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|SQL-Resourceprovider|sqladapter.dbadapter.*&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|MySQL-Resourceprovider|mysqladapter.dbadapter.*&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|App Service|&#42;.appservice.*&lt;region>.&lt;fqdn>*|TCP|80 (HTTP)<br>443 (HTTPS)<br>8172 (MSDeploy)|
|  |&#42;.scm.appservice.*&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)|
|  |api.appservice.*&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)<br>44300 (azure Resource Manager)|
|  |ftp.appservice.*&lt;region>.&lt;fqdn>*|TCP, UDP|21, 1021, 10001-10100 (FTP)<br>990 (FTPS)|
|VPN-gateways|     |     |[Zie de veelgestelde vragen over van de VPN-gateway](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vpn-faq#can-i-traverse-proxies-and-firewalls-using-point-to-site-capability).|
|     |     |     |     |

## <a name="ports-and-urls-outbound"></a>Poorten en URL's (uitgaand)

Azure Stack ondersteunt alleen transparante proxy-servers. In een implementatie waarbij een transparante proxy uplinks met een traditionele proxyserver, moet u toestaan de volgende poorten en URL's voor uitgaande communicatie:

> [!Note]  
> Azure Stack biedt geen ondersteuning voor het bereiken van de Azure-services die worden vermeld in de volgende tabel met Express Route.

|Doel|URL|Protocol|Poorten|
|---------|---------|---------|---------|
|Identiteit|login.windows.net<br>login.microsoftonline.com<br>graph.windows.net<br>https://secure.aadcdn.microsoftonline-p.com<br>Office.com|HTTP<br>HTTPS|80<br>443|
|Marketplace-syndicatie|https://management.azure.com<br>https://&#42;.blob.core.windows.net<br>https://*.azureedge.net<br>https://&#42;.microsoftazurestack.com|HTTPS|443|
|Patch & bijwerken|https://&#42;.azureedge.net|HTTPS|443|
|Registratie|https://management.azure.com|HTTPS|443|
|Gebruik|https://&#42;.microsoftazurestack.com<br>https://*.trafficmanager.NET|HTTPS|443|
|Windows Defender|. wdcp.microsoft.com<br>. wdcpalt.microsoft.com<br>*. updates.microsoft.com<br>*. download.microsoft.com<br>https://msdl.microsoft.com/download/symbols<br>http://www.microsoft.com/pkiops/crl<br>http://www.microsoft.com/pkiops/certs<br>http://crl.microsoft.com/pki/crl/products<br>http://www.microsoft.com/pki/certs<br>https://secure.aadcdn.microsoftonline-p.com<br>|HTTPS|80<br>443|
|NTP|     |UDP|123|
|DNS|     |TCP<br>UDP|53|
|CRL|     |HTTPS|443|
|     |     |     |     |

> [!Note]  
> Uitgaande URL's worden verdeeld met Azure traffic manager voor de best mogelijke connectiviteit op basis van geografische locatie. Met laden met gelijke taakverdeling URL's, Microsoft kunt bijwerken en back-endeindpunten wijzigen zonder gevolgen voor klanten. Microsoft deelt niet de lijst met IP-adressen voor de URL's met taakverdeling. U moet een apparaat dat door de URL in plaats van door IP-filtering ondersteunt.

## <a name="next-steps"></a>Volgende stappen

[Azure Stack-PKI-vereisten](azure-stack-pki-certs.md)
