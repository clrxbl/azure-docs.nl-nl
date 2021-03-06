---
title: Een account voor Cognitive Services API's maken in Azure portal
titlesuffix: Azure Cognitive Services
description: Het maken van een account met Microsoft Cognitive Services API's in Azure portal.
services: cognitive-services
author: garyericson
manager: cgronlun
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 02/01/2018
ms.author: garye
ms.openlocfilehash: 37f53303a3b0c224c1286fb488a796fd5cdee0e5
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386413"
---
# <a name="quickstart-create-a-cognitive-services-account-in-the-azure-portal"></a>Snelstartgids: Een Cognitive Services-account maken in Azure portal

Gebruik deze Quick Start om te beginnen met behulp van Azure Cognitive Services. Deze services worden vertegenwoordigd door Azure [resources](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal), waarmee u verbinding maken met een of meer van de vele Cognitive Services API's beschikbaar.

## <a name="prerequisites"></a>Vereisten

* Een geldig Azure-abonnement. u kunt [maken van een account](https://azure.microsoft.com/free/) gratis.

## <a name="create-and-subscribe-to-an-azure-cognitive-services-resource"></a>Maken en zich abonneren op een Azure Cognitive Services-resource

1. Aanmelden bij de [Azure-portal](http://portal.azure.com), en klikt u op **+ een resource maken**.
    
    ![Cognitive Services API's selecteren](media/cognitive-services-apis-create-account/azurePortalScreen.png)

2. Selecteer onder Azure Marketplace, **AI + Machine Learning**. Als u de service u geïnteresseerd bent in niet ziet, klikt u op **alle** om de volledige catalogus van Cognitive Services API's weer te geven.

    ![Cognitive Services API's selecteren](media/cognitive-services-apis-create-account/azureMarketplace.png)

3. Op de **maken** pagina, geef de volgende informatie:

    |    |    |
    |--|--|
    | **Naam** | Een beschrijvende naam voor uw cognitive services-resource. Wordt u aangeraden een beschrijvende naam, bijvoorbeeld *MyNameFaceAPIAccount*. |
    | **Abonnement** | Selecteer een van uw beschikbare Azure-abonnementen. |
    | **Locatie** | De locatie van uw cognitive service-exemplaar. Verschillende locaties kunnen introduceren latentie, maar hebben geen invloed op de runtime-beschikbaarheid van uw resource. |
    | **Prijscategorie** | De kosten van uw Cognitive Services-account, is afhankelijk van de opties die u kiest en uw gebruik. Zie voor meer informatie, de API [prijsinformatie](https://azure.microsoft.com/pricing/details/cognitive-services/).
    | **Resourcegroep** | De [Azure-resourcegroep](https://docs.microsoft.com/azure/architecture/cloud-adoption/getting-started/azure-resource-access#what-is-an-azure-resource-group) waarin uw Cognitive Services-resource. U kunt een nieuwe groep maken of toe te voegen aan een bestaande groep. |

    ![Maken van het scherm](media/cognitive-services-apis-create-account/resource_create_screen.png)

## <a name="access-your-resource"></a>Toegang tot uw resource 

> [!NOTE]
> Eigenaars van abonnementen het maken van Cognitive Services-accounts voor resourcegroepen en abonnementen kunnen uitschakelen door toe te passen [Azure policy](https://docs.microsoft.com/azure/governance/policy/overview#policy-definition), het toewijzen van een beleidsdefinitie 'niet toegestane resourcetypen' en op te geven **Microsoft.CognitiveServices/accounts** als het doeltype van de resource.

Nadat de resource is gemaakt, kunt u openen vanuit het Azure-dashboard als u deze hebt vastgemaakt. Anders kunt u deze vinden in **resourcegroepen**.

Binnen uw Cognitive Services-resource, kunt u de eindpunt-URL en sleutels in de **overzicht** sectie om te beginnen met het maken van API-aanroepen in uw toepassingen.

![Scherm](media/cognitive-services-apis-create-account/resourceScreen.png)

## <a name="next-steps"></a>Volgende stappen

> [!div class="nextstepaction"]
> [Computer Vision-API C#-zelfstudie](https://docs.microsoft.com/azure/cognitive-services/computer-vision/tutorials/csharptutorial)

## <a name="see-also"></a>Zie ook

* [Snelstartgids: Extraheer handgeschreven tekst uit een installatiekopie](https://docs.microsoft.com/azure/cognitive-services/computer-vision/quickstarts/csharp-hand-text)
* [Zelfstudie: Een app om te detecteren en frame gezichten in een installatiekopie maken](https://docs.microsoft.com/azure/cognitive-services/Face/Tutorials/FaceAPIinCSharpTutorial)
* [Bouw een webpagina aangepast zoeken](https://docs.microsoft.com/azure/cognitive-services/bing-custom-search/tutorials/custom-search-web-page)
* [Language Understanding (LUIS) integreren met een bot met behulp van Bot Framework](https://docs.microsoft.com/azure/cognitive-services/luis/luis-nodejs-tutorial-build-bot-framework-sample)
