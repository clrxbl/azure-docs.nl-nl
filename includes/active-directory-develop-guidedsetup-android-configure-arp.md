---
title: bestand opnemen
description: bestand opnemen
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/13/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: fe2b02b2495b4f37cbc90e1ddbeaca43b41d008c
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843072"
---
## <a name="add-the-applications-registration-to-your-code"></a>Registratie van de toepassing toevoegen aan uw code

In deze stap moet u de toepassing toevoegen / Client-ID aan uw project.

1.  Open `MainActivity` (onder `app`  >  `java`  >  *`{host}.{namespace}`*)
2.  Vervang de regel die begint met `final static String CLIENT_ID`:
```java
final static String CLIENT_ID = "[Enter the application Id here]";
```
3. Open: `app` > `manifests` > `AndroidManifest.xml`
4. Voeg de volgende activiteit in op `manifest\application`. De`BrowserTabActivity` kan aanroepen om uw toepassing na het voltooien van de verificatie van de Microsoft:

```xml
<!--Intent filter to capture System Browser calling back to our app after Sign In-->
<activity
    android:name="com.microsoft.identity.client.BrowserTabActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <!--Add in your scheme/host from registered redirect URI-->
        <!--By default, the scheme should be similar to 'msal[appId]' -->
        <data android:scheme="msal[Enter the application Id here]"
            android:host="auth" />
    </intent-filter>
</activity>
```

