---
title: Powershell gebruiken voor het instellen van waarschuwingen in Application Insights | Microsoft Docs
description: Configuratie van Application Insights om op te halen e-mailberichten over metrische wijzigingen automatiseren.
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 05d6a9e0-77a2-4a35-9052-a7768d23a196
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 10/31/2016
ms.author: mbullwin
ms.openlocfilehash: e7e0f6747beea43b55ae1a20da44cde3c070cd89
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091369"
---
# <a name="use-powershell-to-set-alerts-in-application-insights"></a>PowerShell gebruiken om waarschuwingen in te stellen in Application Insights
U kunt de configuratie van automatiseren [waarschuwingen](app-insights-alerts.md) in [Application Insights](app-insights-overview.md).

Bovendien kunt u [webhooks voor het automatiseren van reacties op een waarschuwing instellen](../monitoring-and-diagnostics/insights-webhooks-alerts.md).

> [!NOTE]
> Als u maken van resources en waarschuwingen op hetzelfde moment wilt, kunt u overwegen [met een Azure Resource Manager-sjabloon](app-insights-powershell.md).
>
>

## <a name="one-time-setup"></a>Eenmalige installatie
Als u PowerShell met uw Azure-abonnement voordat u dit nog niet hebt gebruikt:

Installeer de Azure Powershell-module op de computer waar u de scripts uit te voeren.

* Installeer [Microsoft Web Platform Installer (versie 5 of hoger)](http://www.microsoft.com/web/downloads/platform.aspx).
* Gebruik het Microsoft Azure Powershell installeren

## <a name="connect-to-azure"></a>Verbinding maken met Azure
Open Azure PowerShell en [verbinding maken met uw abonnement](/powershell/azure/overview):

```PowerShell

    Add-AzureAccount
```


## <a name="get-alerts"></a>Waarschuwingen ontvangen
    Get-AzureAlertRmRule -ResourceGroup "Fabrikam" [-Name "My rule"] [-DetailedOutput]

## <a name="add-alert"></a>Waarschuwing toevoegen
    Add-AlertRule  -Name "{ALERT NAME}" -Description "{TEXT}" `
     -ResourceGroup "{GROUP NAME}" `
     -ResourceId "/subscriptions/{SUBSCRIPTION ID}/resourcegroups/{GROUP NAME}/providers/microsoft.insights/components/{APP RESOURCE NAME}" `
     -MetricName "{METRIC NAME}" `
     -Operator GreaterThan  `
     -Threshold {NUMBER}   `
     -WindowSize {HH:MM:SS}  `
     [-SendEmailToServiceOwners] `
     [-CustomEmails "EMAIL1@X.COM","EMAIL2@Y.COM" ] `
     -Location "East US" // must be East US at present
     -RuleType Metric



## <a name="example-1"></a>Voorbeeld 1
Stuur mij een e-mail als reactie op HTTP-aanvragen, meer dan 5 minuten, gemiddelde van de server langzamer dan 1 seconde is. Mijn Application Insights-resource IceCreamWebApp wordt genoemd, en het is in de resourcegroep Fabrikam. Ik ben de eigenaar van het Azure-abonnement.

De GUID is de abonnements-ID (niet de instrumentatiesleutel van de toepassing).

    Add-AlertRule -Name "slow responses" `
     -Description "email me if the server responds slowly" `
     -ResourceGroup "Fabrikam" `
     -ResourceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Fabrikam/providers/microsoft.insights/components/IceCreamWebApp" `
     -MetricName "request.duration" `
     -Operator GreaterThan `
     -Threshold 1 `
     -WindowSize 00:05:00 `
     -SendEmailToServiceOwners `
     -Location "East US" -RuleType Metric

## <a name="example-2"></a>Voorbeeld 2
Ik heb een toepassing die ik gebruik [TrackMetric()](app-insights-api-custom-events-metrics.md#trackmetric) voor het rapporteren van een metrische waarde met de naam "salesPerHour." Stuur dat een e-mail naar Mijn collega's als "salesPerHour" lager dan 100 komt, gemiddelde meer dan 24 uur.

    Add-AlertRule -Name "poor sales" `
     -Description "slow sales alert" `
     -ResourceGroup "Fabrikam" `
     -ResourceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Fabrikam/providers/microsoft.insights/components/IceCreamWebApp" `
     -MetricName "salesPerHour" `
     -Operator LessThan `
     -Threshold 100 `
     -WindowSize 24:00:00 `
     -CustomEmails "satish@fabrikam.com","lei@fabrikam.com" `
     -Location "East US" -RuleType Metric

Dezelfde regel kan worden gebruikt voor de metrische gegevens gerapporteerd met behulp van de [meting parameter](app-insights-api-custom-events-metrics.md#properties) van een andere bijhouden aanroep zoals TrackEvent of trackPageView.

## <a name="metric-names"></a>Metrische namen
| Naam van de meetwaarde | De schermnaam van het | Beschrijving |
| --- | --- | --- |
| `basicExceptionBrowser.count` |Browseruitzonderingen |Aantal niet-onderschepte uitzonderingen in de browser. |
| `basicExceptionServer.count` |Serveruitzonderingen |Aantal onverwerkte uitzonderingen in de app. |
| `clientPerformance.clientProcess.value` |Verwerkingstijd client |Tijd tussen ontvangst van de laatste byte van een document totdat de DOM wordt geladen. Asynchrone aanvragen kunnen nog worden verwerkt. |
| `clientPerformance.networkConnection.value` |Netwerkverbindingstijd voor laden van pagina |Tijd voor de browser verbinding maken met het netwerk. Kan zijn ingesteld op 0 als in de cache. |
| `clientPerformance.receiveRequest.value` |Reactietijd voor ontvangen |Tijd tussen de browser aanvraag te verzenden naar het begint met het antwoord kan ontvangen. |
| `clientPerformance.sendRequest.value` |Verzoektijd voor verzenden |Tijd die door de browser om aanvraag te verzenden. |
| `clientPerformance.total.value` |Laadtijd van browserpagina |Tijd vanaf gebruikersaanvraag totdat DOM, opmaakmodellen, scripts en afbeeldingen zijn geladen. |
| `performanceCounter.available_bytes.value` |Beschikbaar geheugen |Fysiek geheugen is onmiddellijk beschikbaar voor een proces of voor systeemgebruik. |
| `performanceCounter.io_data_bytes_per_sec.value` |I/O-snelheid van proces |Totaal aantal per seconde gelezen en naar bestanden, netwerk en apparaten geschreven bytes. |
| `performanceCounter.number_of_exceps_thrown_per_sec.value` |aantal uitzonderingen |Uitzonderingen per seconde. |
| `performanceCounter.percentage_processor_time.value` |CPU voor proces |Het percentage verstreken tijd van alle procesthreads is gebruikt door de processor instructies voor het proces van toepassingen. |
| `performanceCounter.percentage_processor_total.value` |Processortijd |Het tijdspercentage dat de processor spendeert aan niet-inactieve threads. |
| `performanceCounter.process_private_bytes.value` |Proceseigen bytes |Geheugen exclusief toegewezen aan de processen van de gecontroleerde toepassing. |
| `performanceCounter.request_execution_time.value` |Uitvoeringstijd voor ASP.NET-aanvraag |Uitvoeringstijd van de meest recente aanvraag. |
| `performanceCounter.requests_in_application_queue.value` |ASP.NET aanvragen in wachtrij voor uitvoering |Lengte van de rij met toepassingsaanvragen. |
| `performanceCounter.requests_per_sec.value` |Snelheid van aanvragen voor ASP.NET |Snelheid per seconde van alle aanvragen aan de toepassing vanaf ASP.NET. |
| `remoteDependencyFailed.durationMetric.count` |Afhankelijkheidsfouten |Het aantal mislukte aanroepen van de server-toepassing naar externe bronnen. |
| `request.duration` |Serverreactietijd |Tijd tussen de ontvangst van een HTTP-aanvraag en het voltooien van het verzenden van de reactie. |
| `request.rate` |Aanvraagsnelheid |Snelheid per seconde van alle aanvragen aan de toepassing. |
| `requestFailed.count` |Mislukte aanvragen |Aantal HTTP-aanvragen dat heeft geresulteerd in een responscode > = 400 |
| `view.count` |Paginaweergaven |Het aantal aanvragen van clientgebruikers voor webpagina's. Synthetisch verkeer is gefilterd. |
| {uw eigen aangepaste metrische naam} |{De naam van de meetwaarde} |De waarde van de metrische gegevens die zijn gerapporteerd door [TrackMetric](app-insights-api-custom-events-metrics.md#trackmetric) of in de [metingen parameter van de aanroep van een tracering](app-insights-api-custom-events-metrics.md#properties). |

De metrische gegevens worden verzonden door de verschillende telemetrie-modules:

| Metrische groep | Collector-module |
| --- | --- |
| basicExceptionBrowser,<br/>clientPerformance,<br/>weergeven |[Browser JavaScript](app-insights-javascript.md) |
| performanceCounter |[Prestaties](app-insights-configuration-with-applicationinsights-config.md) |
| remoteDependencyFailed |[Afhankelijkheid](app-insights-configuration-with-applicationinsights-config.md) |
| -aanvraag<br/>requestFailed |[Serveraanvraag](app-insights-configuration-with-applicationinsights-config.md) |

## <a name="webhooks"></a>Webhooks
U kunt [automatiseren van reacties op een waarschuwing](../monitoring-and-diagnostics/insights-webhooks-alerts.md). Azure roept een webadres van uw keuze wanneer een waarschuwing wordt gegenereerd.

## <a name="see-also"></a>Zie ook
* [Script voor het configureren van Application Insights](app-insights-powershell-script-create-resource.md)
* [Application Insights en test bronnen op het web maken met sjablonen](app-insights-powershell.md)
* [Automatiseren met Microsoft Azure Diagnostics naar Application Insights koppelen](app-insights-powershell-azure-diagnostics.md)
* [Uw reactie op een waarschuwing automatiseren](../monitoring-and-diagnostics/insights-webhooks-alerts.md)
