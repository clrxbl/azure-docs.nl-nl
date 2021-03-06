---
title: Problemen met ontbrekende gegevens oplossen - Application Insights voor .NET
description: Niet al gegevens in Azure Application Insights? Probeer het hier.
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: e231569f-1b38-48f8-a744-6329f41d91d3
ms.service: application-insights
ms.workload: mobile
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 07/23/2018
ms.author: mbullwin
ms.openlocfilehash: 1a46564c324edb1999a2e1b1d482817685df2893
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/23/2018
ms.locfileid: "39205983"
---
# <a name="troubleshooting-no-data---application-insights-for-net"></a>Problemen met ontbrekende gegevens oplossen - Application Insights voor .NET
## <a name="some-of-my-telemetry-is-missing"></a>Aantal van mijn telemetrie ontbreekt
*In Application Insights Zie ik slechts een fractie van de gebeurtenissen die worden gegenereerd door mijn app.*

* Als u het gedeelte van het dezelfde consistent ziet, is het waarschijnlijk vanwege adaptieve [steekproeven](app-insights-sampling.md). U kunt dit controleren zoeken (op de overzichtsblade) openen en zoek naar een exemplaar van een aanvraag of een andere gebeurtenis. Aan de onderkant van de sectie met eigenschappen klikt u op '...' voor volledige informatie. Als aantal > 1 aanvragen en vervolgens steekproeven worden uitgevoerd. 
* Anders is het mogelijk dat u ondervindt een [limiet](app-insights-pricing.md#limits-summary) voor uw prijsplan. Deze limieten gelden per minuut.

## <a name="no-data-from-my-server"></a>Er zijn geen gegevens van mijn server
*Ik mijn app hebt geïnstalleerd op de webserver en nu kan ik geen telemetriegegevens van deze niet ziet. Het is gegaan OK op mijn dev-machine.*

* Waarschijnlijk een probleem met firewall. [Instellen van firewalluitzonderingen voor Application Insights om gegevens te verzenden](app-insights-ip-addresses.md).
* IIS-Server mogelijk ontbreken bepaalde vereisten: .NET-Extensibility 4.5 en ASP.NET 4.5.

*Ik [geïnstalleerd Status Monitor](app-insights-monitor-performance-live-website-now.md) op de webserver voor het bewaken van bestaande apps. Geen resultaten weergegeven niet.*

* Zie [statusmonitor oplossen](app-insights-monitor-performance-live-website-now.md#troubleshooting-runtime-configuration-of-application-insights). 

## <a name="q01"></a>Geen optie 'Application Insights toevoegen' in Visual Studio
*Wanneer ik met de rechtermuisknop op een bestaand project in Solution Explorer, weergegeven niet alle opties voor Application Insights.*

* Niet alle typen .NET-project worden ondersteund door de hulpprogramma's. Web- en WCF-projecten worden ondersteund. Voor andere projecttypen zoals toepassingen voor desktop- of service, kunt u nog steeds [een Application Insights SDK handmatig toevoegen aan uw project](app-insights-windows-desktop.md).
* Zorg ervoor dat u hebt [Visual Studio 2013 Update 3 of hoger](https://docs.microsoft.com/visualstudio/releasenotes/vs2013-update3-rtm-vs). Dit is vooraf geïnstalleerd met analyses voor ontwikkelaars-hulpprogramma's, waardoor de Application Insights-SDK.
* Selecteer **extra**, **extensies en Updates** en controleert u of **Developer Analytics Tools** is geïnstalleerd en ingeschakeld. Als dit het geval is, klikt u op **Updates** om te zien of er een update beschikbaar is.
* Open het dialoogvenster Nieuw Project en kies ASP.NET-webtoepassing. Als u de optie van de Application Insights ziet, klikt u vervolgens de hulpprogramma's geïnstalleerd. Als dat niet het geval is, probeert te verwijderen en vervolgens opnieuw de Application Insights-hulpprogramma's te installeren.

## <a name="q02"></a>Application Insights toevoegen is mislukt
*Wanneer ik Application Insights toevoegen aan een bestaand project, zie ik een foutbericht weergegeven.*

Mogelijke oorzaken:

* Communicatie met de Application Insights-portal is mislukt; of
* Er is een probleem met uw Azure-account;
* U hebt alleen [leestoegang tot het abonnement of de groep waar u probeert te maken van de nieuwe resource](app-insights-resources-roles-access-control.md).

FIX:

* Controleer dat u hebt opgegeven aanmeldingsreferenties voor de juiste Azure-account. 
* In uw browser, Controleer of u toegang tot hebt de [Azure-portal](https://portal.azure.com). Instellingen openen en kijk of er verder geen beperkingen.
* [Application Insights toevoegen aan een bestaand project](app-insights-asp-net.md): In Solution Explorer, klik met de rechtermuisknop op uw project en kies 'Application Insights toevoegen'.
* Als deze nog steeds niet werkt, volgt u de [handmatige procedure](app-insights-windows-services.md) toevoegen van een resource in de portal en vervolgens de SDK toevoegen aan uw project. 

## <a name="emptykey"></a>Ik krijg de foutmelding 'instrumentatiesleutel mag niet leeg zijn'
Er is iets fout gegaan tijdens installatie Application Insights of misschien een adapter voor logboekregistratie.

Klik in Solution Explorer met de rechtermuisknop op uw project en kies **Application Insights > Application Insights configureren**. U krijgt een dialoogvenster waarin u zich aanmeldt bij Azure uitnodigt en maak een Application Insights-resource, of gebruik een bestaande resourcegroep.

## <a name="NuGetBuild"></a> "NuGet-pakketten ontbreken' op mijn buildserver
*Alles bouwt OK wanneer ik ben foutopsporing op de ontwikkelcomputer, maar ik krijg een NuGet-fout op de buildserver.*

Raadpleeg [NuGet-pakket herstellen](http://docs.nuget.org/Consume/Package-Restore) en [automatisch herstellen van pakket](http://docs.nuget.org/Consume/package-restore/migrating-to-automatic-package-restore).

## <a name="missing-menu-command-to-open-application-insights-from-visual-studio"></a>Ontbrekende opdracht Application Insights openen vanuit Visual Studio
*Wanneer ik mijn project Solution Explorer met de rechtermuisknop, zie ik geen Application Insights-opdrachten of ik een Open Application Insights-opdracht niet wordt weergegeven.*

Mogelijke oorzaken:

* Als u de Application Insights-resource die handmatig zijn gemaakt, of als het project is van een type dat wordt niet ondersteund door de Application Insights-hulpprogramma's.
* De hulpprogramma's voor analyses voor ontwikkelaars zijn uitgeschakeld in uw Visual Studio. 
* Uw Visual Studio is ouder dan 2013 Update 3.

FIX:

* Zorg ervoor dat uw versie van Visual Studio 2013 update 3 of hoger.
* Selecteer **extra**, **extensies en Updates** en controleert u of **Developer Analytics tools** is geïnstalleerd en ingeschakeld. Als dit het geval is, klikt u op **Updates** om te zien of er een update beschikbaar is.
* Met de rechtermuisknop op uw project in Solution Explorer. Als u de opdracht ziet **Application Insights > Application Insights configureren**, gebruiken om te verbinden met uw project de resource in de Application Insights-service.

Anders, dat het projecttype rechtstreeks wordt niet ondersteund door de Application Insights-hulpprogramma's. Als u wilt uw telemetrie weergeven, moet u zich aanmelden bij de [Azure-portal](https://portal.azure.com), kiest u de Application Insights in de linker navigatiebalk en selecteer uw toepassing.

## <a name="access-denied-on-opening-application-insights-from-visual-studio"></a>'Toegang geweigerd' op de Application Insights openen vanuit Visual Studio
*De opdracht 'Open Application Insights' kom ik bij de Azure-portal, maar ik krijg de foutmelding 'toegang geweigerd'.*

![](./media/app-insights-asp-net-troubleshoot-no-data/access-denied.png)

De aanmelding voor Microsoft die u hebt voor het laatst op uw standaardbrowser gebruikt heeft geen toegang tot [de resource die is gemaakt bij de Application Insights is toegevoegd aan deze app](app-insights-asp-net.md). Er zijn twee waarschijnlijke oorzaken: 

* U hebt meer dan één Microsoft-account - misschien een werk- en een persoonlijk Microsoft-account? De aanmelding die u hebt voor het laatst op uw standaardbrowser gebruikt is voor een ander account dan het account dat toegang tot heeft [Application Insights toevoegen aan het project](app-insights-asp-net.md). 
  
  * FIX: Klik op de naam op van uw rechtsboven in het browservenster en meld u af. Meld u vervolgens aan met het account dat toegang heeft. Klik vervolgens op de linker navigatiebalk op Application Insights en selecteer uw app.
* Iemand anders Application Insights hebt toegevoegd aan het project en ze bent vergeten om u te bieden [toegang tot de resourcegroep](app-insights-resources-roles-access-control.md) waarin deze is gemaakt. 
  
  * Oplossing: Als ze een organisatie-account gebruikt, ze kunnen u toevoegen aan het team. of ze kunnen u afzonderlijke toegang verlenen tot de resourcegroep.

## <a name="asset-not-found-on-opening-application-insights-from-visual-studio"></a>'Asset niet gevonden' op de Application Insights openen vanuit Visual Studio
*De opdracht 'Open Application Insights' kom ik bij de Azure-portal, maar ik krijg de foutmelding 'asset is niet gevonden'.*

Mogelijke oorzaken:

* De Application Insights-resource voor uw toepassing is verwijderd; of
* De instrumentatiesleutel is ingesteld of gewijzigd in ApplicationInsights.config door rechtstreeks te bewerken zonder het projectbestand worden bijgewerkt. 

De instrumentatiesleutel in ApplicationInsights.config besturingselementen waarbij de telemetrie wordt verzonden. Een regel in het projectbestand bepaalt welke resource wordt geopend wanneer u de opdracht in Visual Studio gebruiken. 

FIX:

* Klik in Solution Explorer met de rechtermuisknop op het project en kies Application Insights, Application Insights configureren. U kunt in het dialoogvenster, ofwel telemetrie wordt verzonden naar een bestaande resource of een nieuwe maken. Of:
* Open de resource direct. Aanmelden bij [de Azure-portal](https://portal.azure.com), klikt u op Application Insights in de linker navigatiebalk en selecteer vervolgens de app.

## <a name="where-do-i-find-my-telemetry"></a>Waar vind ik mijn telemetrie?
*Ik aangemeld bij de [Microsoft Azure portal](https://portal.azure.com), en ik ben kijken naar de Azure home-dashboard. Dus waar vind ik mijn gegevens Application Insights?*

* Klik in de linker navigatiebalk op Application Insights en vervolgens de appnaam van uw. Als u alle projecten er geen hebt, moet u [toevoegen of configureren van Application Insights in uw webproject](app-insights-asp-net.md).
  
    Daar ziet u enkele samenvatting grafieken. U kunt klikken voor meer details door.
* Klik op de knop Application Insights in Visual Studio, tijdens de foutopsporing van uw app.

## <a name="q03"></a> Er is geen server-gegevens (of geen gegevens op alle)
*Kan ik mijn app wordt uitgevoerd en vervolgens de Application Insights-service opent in Microsoft Azure, maar alle grafieken weergeven 'Meer informatie over het verzamelen van...' of 'Niet geconfigureerd.'* Of, *alleen paginaweergave en gebruikersgegevens, maar geen servergegevens.*

* Voer uw toepassing uit in de foutopsporingsmodus in Visual Studio (F5). Gebruik de toepassing om telemetrie te genereren. Controleer of u gebeurtenissen vastgelegd in het uitvoervenster van Visual Studio kunt zien. 
  
    ![](./media/app-insights-asp-net-troubleshoot-no-data/output-window.png)
* Open in de Application Insights-webportal [diagnostische gegevens doorzoeken](app-insights-diagnostic-search.md). Gegevens weergegeven meestal hier eerst.
* Klik op de knop vernieuwen. De blade wordt vernieuwd periodiek, maar u kunt ook dit handmatig doen. Het interval voor vernieuwen is langer voor grotere tijdsbereik.
* Controleer dat de instrumentatiesleutels overeenkomen. Op de hoofdblade voor uw app in de Application Insights-portal, in de **Essentials** vervolgkeuzelijst, kijkt u naar **instrumentatiesleutel**. Vervolgens in uw project in Visual Studio, opent u ApplicationInsights.config en zoek de `<instrumentationkey>`. Controleer of de twee sleutels gelijk zijn. Als dit niet het:
  
  * In de portal, klikt u op de Application Insights en zoek naar de app-resource met de juiste sleutel; of
  * In Visual Studio Solution Explorer met de rechtermuisknop op het project en kies Application Insights kunt configureren. Opnieuw instellen van de app om telemetrie te verzenden naar de juiste bron.
  * Als u de bijbehorende sleutels niet kunt vinden, controleer dan of u dezelfde referenties aanmelden in Visual Studio als in de portal.
    
    ![](./media/app-insights-asp-net-troubleshoot-no-data/ikey-check.png)
* In de [home Microsoft Azure-dashboard](https://portal.azure.com), bekijk de servicestatus-kaart. Als er enkele waarschuwingen indicaties zijn, wacht u totdat ze weer terug bent op OK en vervolgens sluiten en uw Application Insights-toepassing-blade opnieuw open.
* Controleer ook [onze blog status](http://blogs.msdn.com/b/applicationinsights-status/).
* Hebt u een code te schrijven voor de [server-side SDK](app-insights-api-custom-events-metrics.md) die veranderen de instrumentatiesleutel in `TelemetryClient` exemplaren of in `TelemetryContext`? Of hebt u schrijft een [filter of steekproeven](app-insights-api-filtering-sampling.md) die kan worden gefilterd te veel?
* Als u ApplicationInsights.config hebt bewerkt, zorgvuldig controleert u de configuratie van [TelemetryInitializers en TelemetryProcessors](app-insights-api-filtering-sampling.md). Een verkeerde naam type of de parameter kan leiden tot de SDK om geen gegevens te verzenden.

## <a name="q04"></a>Er zijn geen gegevens over het gebruik van paginaweergaven, Browsers,
*Ik zie de gegevens in serverreactietijd en serveraanvragen grafieken, maar er zijn geen gegevens in de weergave paginalaadtijd of op de blades Browser en het gebruik.*

De gegevens afkomstig van scripts in de webpagina's. 

* Als u Application Insights hebt toegevoegd aan een bestaand webproject [hebt tot de scripts handmatig toevoegen](app-insights-javascript.md).
* Zorg ervoor dat Internet Explorer uw site wordt niet weergegeven in de compatibiliteitsmodus.
* Functie voor foutopsporing van de browser (F12 op sommige browsers, kies vervolgens netwerk) gebruiken om te controleren of dat gegevens worden verzonden naar `dc.services.visualstudio.com`.

## <a name="no-dependency-or-exception-data"></a>Er zijn geen gegevens afhankelijkheid of uitzondering
Zie [afhankelijkheidstelemetrie](app-insights-asp-net-dependencies.md) en [uitzonderingstelemetrie](app-insights-asp-net-exceptions.md).

## <a name="no-performance-data"></a>Er zijn geen prestatiegegevens
Prestatiegegevens (CPU, IO-snelheid, enzovoort) is beschikbaar voor [Java-web-services](app-insights-java-collectd.md), [Windows-bureaublad-apps](app-insights-windows-desktop.md), [IIS web-apps en services als u statusmonitor installeert](app-insights-monitor-performance-live-website-now.md), en [Azure Cloudservices](app-insights-azure.md). u vindt deze onder instellingen voor Servers.

## <a name="no-server-data-since-i-published-the-app-to-my-server"></a>Er zijn geen gegevens (server) sinds ik de app is gepubliceerd naar Mijn server
* Controleer dat u daadwerkelijk alle Microsoft gekopieerd. Application Insights-dll-bestanden naar de server, samen met Microsoft.Diagnostics.Instrumentation.Extensions.Intercept.dll
* In de firewall, u mogelijk [bepaalde TCP-poorten openen](app-insights-ip-addresses.md).
* Hebt u een proxyserver gebruikt voor het verzenden van buiten uw bedrijfsnetwerk, stelt u de [defaultProxy](https://msdn.microsoft.com/library/aa903360.aspx) in Web.config
* WindowsServer 2008: Zorg ervoor dat u de volgende updates hebt geïnstalleerd: [KB2468871](https://support.microsoft.com/kb/2468871), [KB2533523](https://support.microsoft.com/kb/2533523), [KB2600217](https://support.microsoft.com/kb/2600217).

## <a name="i-used-to-see-data-but-it-has-stopped"></a>Ik gebruikt om gegevens te bekijken, maar deze is gestopt
* Controleer de [status blog](http://blogs.msdn.com/b/applicationinsights-status/).
* Hebt u uw maandelijkse quotum van gegevenspunten bereikt? Open de instellingen/quotum en prijzen om erachter te komen. Als dit het geval is, kunt u uw abonnement upgraden of betalen voor extra capaciteit. Zie de [prijzen schema](https://azure.microsoft.com/pricing/details/application-insights/).

## <a name="i-dont-see-all-the-data-im-expecting"></a>Ik zie niet alle gegevens die ik verwacht
Als uw toepassing grote hoeveelheden gegevens verzendt en u de Application Insights-SDK voor ASP.NET-versie 2.0.0-beta3 of hoger, gebruikt de [adaptieve steekproeven](app-insights-sampling.md) functie kan werken en slechts een percentage van uw telemetrie te verzenden. 

U kunt deze uitschakelen, maar dit wordt niet aanbevolen. Steekproeven is zo ontworpen dat gerelateerde telemetrie correct wordt verzonden, voor diagnostische doeleinden. 

## <a name="client-ip-address-is-0000"></a>Client-IP-adres is 0.0.0.0 
Februari 2018, we [aangekondigd](https://blogs.msdn.microsoft.com/applicationinsights-status/2018/02/01/all-octets-of-ip-address-will-be-set-to-zero/) dat we het vastleggen van de Client-IP-adres verwijderd. Dit heeft geen invloed op geografische locatie.

## <a name="wrong-geographical-data-in-user-telemetry"></a>Onjuiste geografische gegevens in de telemetrie van de gebruiker
De plaats, regio en land dimensies zijn afgeleid van IP-adressen en niet altijd nauwkeurig. Deze IP-adressen zijn voor de locatie als eerste verwerkt en vervolgens worden gewijzigd op 0.0.0.0 worden opgeslagen.

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>Uitzondering Methode niet gevonden bij het uitvoeren van uw app in Azure Cloud Services
Hebt u uw app ontwikkeld voor .NET 4.6? 4.6 wordt niet automatisch ondersteund in Azure Cloud Services-rollen. [Installeer 4.6 voor elke rol](../cloud-services/cloud-services-dotnet-install-dotnet.md) voordat u uw app uitvoert.

## <a name="still-not-working"></a>Nog steeds werkt niet...
* [Application Insights-forum](https://social.msdn.microsoft.com/Forums/vstudio/en-US/home?forum=ApplicationInsights)

