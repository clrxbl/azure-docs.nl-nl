---
title: Resource en de sleutel-management - QnA Maker
titleSuffix: Azure Cognitive Services
description: Uw service QnA Maker omgaat met twee soorten sleutels, abonnementssleutels en eindpunt sleutels.
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: f48b4b7da4990de0efb9b7de5d97731d748f0e7f
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040445"
---
# <a name="key-management"></a>Sleutelbeheer

Uw service QnA Maker omgaat met twee soorten sleutels, **abonnementssleutels** en **eindpunt sleutels**.

![Sleutelbeheer](../media/qnamaker-how-to-key-management/key-management.png)

1. **Abonnementssleutels**: deze sleutels worden gebruikt voor toegang tot de [QnA Maker management-service API's](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff). Deze API's kunt u verschillende CRUD-bewerkingen op uw knowledge base uitvoeren.  

2. **Eindpunt sleutels**: deze sleutels worden gebruikt voor toegang tot het eindpunt van de knowledge base voor een antwoord voor de vraag van een gebruiker. Doorgaans gebruikt u dit eindpunt in de chat bot/App-code die de QnA Maker-service worden verbruikt.
 
## <a name="subscription-keys"></a>Abonnementssleutels
U kunt weergeven en opnieuw instellen van uw abonnementssleutels vanuit de Azure-portal waar u de QnA Maker-resource hebt gemaakt. 
1. Ga naar de QnA Maker-resource in Azure portal.

    ![Lijst met resources voor QnA Maker](../media/qnamaker-how-to-key-management/qnamaker-resource-list.png)

2. Ga naar **sleutels**.

    ![Abonnementssleutel](../media/qnamaker-how-to-key-management/subscription-key.PNG)

## <a name="endpoint-keys"></a>Eindpunt-sleutels

Eindpunt-sleutels kunnen worden beheerd vanuit de [QnA Maker portal](https://qnamaker.ai).

1. Meld u aan bij de [QnA Maker portal](https://qnamaker.ai), en Ga naar **sleutels beheren**.

    ![Eindpuntsleutel](../media/qnamaker-how-to-key-management/Endpoint-keys.png)

2. Weergeven of opnieuw instellen van uw sleutels.

    ![belangrijkste endpoint-beheerder](../media/qnamaker-how-to-key-management/Endpoint-keys1.png)

    >[!NOTE]
    >Als u denkt dat ze zijn aangetast, vernieuwt u uw sleutels. Dit is mogelijk bijbehorende wijzigingen in uw App-Bot-code.

## <a name="next-steps"></a>Volgende stappen

> [!div class="nextstepaction"]
> [Een kennisdatabase maken in een andere taal](./language-knowledge-base.md)
