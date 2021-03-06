---
title: De Azure Active Directory-app-manifest begrijpen | Microsoft Docs
description: Gedetailleerde dekking van de app-manifest van Azure Active Directory, die staat voor configuratie van de identiteit van een toepassing in een Azure AD-tenant, en wordt gebruikt om toegang te vergemakkelijken OAuth autorisatie, toestemming ervaring en meer.
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 4804f3d4-0ff1-4280-b663-f8f10d54d184
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.reviewer: sureshja
ms.openlocfilehash: bc7999d56da8398b4f54b0144a595ee7c2e2ea35
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/11/2018
ms.locfileid: "49115107"
---
# <a name="azure-active-directory-app-manifest"></a>Azure Active Directory-app-manifest

Apps die met Azure Active Directory (Azure AD integreren) moeten zijn geregistreerd met een Azure AD-tenant. U kunt de app in configureren de [Azure-portal](https://portal.azure.com) door te selecteren **App-registraties** onder **Azure Active Directory**, kiezen van de app die u configureren wilt, en vervolgens selecteren **Manifest**.

## <a name="manifest-reference"></a>Naslaginformatie over het manifest

> [!NOTE]
> Als u niet zien de **Voorbeeldwaarde** kolom na de **beschrijving**, het browservenster maximaliseren en schuiven/Veeg tot u ziet de **Voorbeeldwaarde** kolom.

>[!div class="mx-tdBreakAll"]
>[!div class="mx-tdCol2BreakAll"]

| Sleutel  | Waardetype | Beschrijving  | Voorbeeldwaarde |
|---------|---------|---------|---------|
| `accessTokenAcceptedVersion` | Null-waarden Int32 | Hiermee geeft u de geaccepteerde access token versie voor de huidige API-resource. Mogelijke waarden zijn 1, 2, null. Standaard ingesteld op null. dit wordt beschouwd als 2. | `2` |
| `allowPublicClient` | booleaans | Hiermee geeft u de alternatieve toepassingstype. Azure AD bepaalt het type van de replyUrlsWithType standaard. Er zijn bepaalde scenario's waar Azure AD het client-app-type kan niet bepalen (bijvoorbeeld [ROPC](https://tools.ietf.org/html/rfc6749#section-4.3) stroom waar de HTTP-aanvraag zonder een URL-omleiding plaatsvindt). In deze gevallen wordt het toepassingstype op basis van de waarde van deze eigenschap worden geïnterpreteerd door Azure AD. Als deze waarde is ingesteld op waar het alternatieve toepassingstype is ingesteld als openbare client, zoals een geïnstalleerde app die wordt uitgevoerd op een mobiel apparaat. De standaardwaarde is false, wat inhoudt dat de terugval toepassingstype vertrouwelijke client, zoals web-app. | `false` |
| `appId` | Id-reeks | Hiermee geeft u de unieke id voor de app die is toegewezen aan een app door Azure AD. | `"601790de-b632-4f57-9523-ee7cb6ceba95"` |
| `appRoles` | Het type matrix | Hiermee geeft u de verzameling toepassingsrollen die mogelijk zijn gedeclareerd in een app. Deze rollen kunnen worden toegewezen aan gebruikers, groepen of service-principals. Zie voor meer voorbeelden en informatie [app-rollen in uw toepassing toevoegen en deze ontvangen in het token](howto-add-app-roles-in-azure-ad-apps.md) | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"allowedMemberTypes": [<br>&emsp;&nbsp;&nbsp;&nbsp;"User"<br>&nbsp;&nbsp;&nbsp;],<br>&nbsp;&nbsp;&nbsp;"description":"Read-only access to device information",<br>&nbsp;&nbsp;&nbsp;"displayName":"Read Only",<br>&nbsp;&nbsp;&nbsp;"id":guid,<br>&nbsp;&nbsp;&nbsp;"isEnabled":true,<br>&nbsp;&nbsp;&nbsp;"value":"ReadOnly"<br>&nbsp;&nbsp;}<br>]</code>  |
| `groupMembershipClaims` | tekenreeks | Een bitmasker dat Hiermee configureert u de `groups` claim uitgegeven in een OAuth 2.0-toegangstoken dat de app wordt verwacht dat of de gebruiker. De Bitmaskerwaarden zijn:<br>0: geen<br>1: beveiligingsgroepen en Azure AD-rollen<br>2: gereserveerd<br>4: gereserveerd<br>Het bitmasker instelt op 7 krijgt alle van de beveiligingsgroepen, distributiegroepen en Azure AD-maprollen die de aangemelde gebruiker lid van is. | `1` |
| `optionalClaims` | tekenreeks | De optionele claims die in het token wordt geretourneerd door de service voor beveiligingstokens voor deze specifieke app.<br>Op dit moment niet de apps die ondersteuning bieden voor zowel persoonlijke accounts en Azure AD (geregistreerd via de portal van de registratie van de app) optioneel claims gebruiken. Apps die zijn geregistreerd voor alleen Azure AD met behulp van het v2.0-eindpunt kunnen beschikt echter over de optionele claims die ze in het manifest wordt aangevraagd. Zie voor meer informatie, [optionele claims](active-directory-optional-claims.md). | `null` |
| `id` | Id-reeks | De unieke id voor de app in de map. Deze ID is niet de id die wordt gebruikt om de app in een transactie protocol te identificeren. Het wordt gebruikt voor de verwijzing naar het object in de directory-query's. | `"f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"` |
| `identifierUris` | String-matrix | Gebruiker gedefinieerde URI(s) unieke identificatie van een Web-app binnen de Azure AD-tenant of binnen een gecontroleerd aangepast domein als de app met meerdere tenants. | <code>[<br>&nbsp;&nbsp;"https://MyRegistererdApp"<br>]</code> |
| `informationalUrls` | tekenreeks | Hiermee geeft u de koppelingen met de gebruiksvoorwaarden en privacyverklaring van de app. De gebruiksvoorwaarden en privacyverklaring worden opgehaald voor gebruikers via de ervaring van de gebruiker toestemming. Zie voor meer informatie, [hoe: gebruiksvoorwaarden en privacyverklaring toevoegen voor Azure AD-apps geregistreerd](howto-add-terms-of-service-privacy-statement.md). | <code>{<br>&nbsp;&nbsp;&nbsp;"marketing":"https://MyRegisteredApp/marketing",<br>&nbsp;&nbsp;&nbsp;"privacy":"https://MyRegisteredApp/privacystatement",<br>&nbsp;&nbsp;&nbsp;"support":"https://MyRegisteredApp/support",<br>&nbsp;&nbsp;&nbsp;"termsOfService":"https://MyRegisteredApp/termsofservice"<br>}</code> |
| `keyCredentials` | Het type matrix | Bevat verwijzingen naar referenties app is toegewezen, gedeelde geheimen op basis van een tekenreeks en X.509-certificaten. Deze referenties worden gebruikt bij het aanvragen van toegangstokens (wanneer de app fungeert als een client in plaats van die als bron). | <code>[<br>&nbsp;{<br>&nbsp;&nbsp;&nbsp;"customKeyIdentifier":null,<br>&nbsp;&nbsp;&nbsp;"endDate":"2018-09-13T00:00:00Z",<br>&nbsp;&nbsp;&nbsp;"keyId":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"startDate":"2017-09-12T00:00:00Z",<br>&nbsp;&nbsp;&nbsp;"type":"AsymmetricX509Cert",<br>&nbsp;&nbsp;&nbsp;"usage":"Verify",<br>&nbsp;&nbsp;&nbsp;"value":null<br>&nbsp;&nbsp;}<br>]</code> |
| `knownClientApplications` | Het type matrix | Gebruikt voor bundelen toestemming hebt u een oplossing die bestaat uit twee delen: een client-app en een aangepaste web-API-app. Als u de toepassings-id van de client-app in deze waarde invoert, wordt alleen de gebruiker toe te staan wanneer de client-app hebben. Azure AD weet dat ermee akkoord dat de client betekent impliciet stemt ermee in dat de web-API en service-principals voor de client en de web-API wordt automatisch worden ingericht op hetzelfde moment. De client en de web-API-app moeten worden geregistreerd in dezelfde tenant. | `[GUID]` |
| `logoUrl` | tekenreeks | Alleen de waarde die verwijst naar de URL van het CDN logo dat is geüpload in de portal lezen. | `https://MyRegisteredAppLogo` |
| `logoutUrl` | tekenreeks | De URL moet zich afmelden bij de app. | `https://MyRegisteredAppLogout` |
| `name` | tekenreeks | De weergavenaam voor de app. | `MyRegisteredApp` |
| `oauth2AllowImplicitFlow` | booleaans | Hiermee geeft u op of deze web-app toegangstokens voor OAuth 2.0-impliciete stroom kan opvragen. De standaardwaarde is false. Met deze markering wordt gebruikt voor de browser gebaseerde apps, zoals Javascript-apps van één pagina. Voor meer informatie, voer `OAuth 2.0 implicit grant flow` in de inhoudsopgave en Zie de onderwerpen over de impliciete stroom. | `false` |
| `oauth2AllowIdTokenImplicitFlow` | booleaans | Hiermee geeft u op of deze web-app ID-tokens van OAuth 2.0-impliciete stroom kan opvragen. De standaardwaarde is false. Met deze markering wordt gebruikt voor de browser gebaseerde apps, zoals Javascript-apps van één pagina. | `false` |
| `oauth2Permissions` | Het type matrix | Hiermee geeft u het verzamelen van OAuth 2.0-machtigingsbereiken waarmee de web-API (resource)-app beschikbaar wordt gemaakt op client-apps. Dit bereik aan machtigingen kunnen tijdens toestemming op client-apps worden verleend. | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"adminConsentDescription":"Allow the app to access resources on behalf of the signed-in user.",<br>&nbsp;&nbsp;&nbsp;"adminConsentDisplayName":"Access resource1",<br>&nbsp;&nbsp;&nbsp;"id":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"isEnabled":true,<br>&nbsp;&nbsp;&nbsp;"type":"User",<br>&nbsp;&nbsp;&nbsp;"userConsentDescription":"Allow the app to access resource1 on your behalf.",<br>&nbsp;&nbsp;&nbsp;"userConsentDisplayName":"Access resources",<br>&nbsp;&nbsp;&nbsp;"value":"u| Hiermee geeft u het verzamelen van OAuth 2.0-machtigingsbereiken waarmee de web-API (resource)-app beschikbaar wordt gemaakt op client-apps. Dit bereik aan machtigingen kunnen tijdens toestemming op client-apps worden verleend. ser_impersonation"<br>&nbsp;&nbsp;}<br>]</code> |
| `oauth2RequiredPostResponse` | booleaans | Hiermee geeft u op of, als onderdeel van OAuth 2.0 token aanvragen, Azure AD POST-aanvragen, in plaats van GET-aanvragen kunnen. De standaardwaarde is ingesteld op false, dat aangeeft dat alleen GET-aanvragen kunnen worden. | `false` |
| `parentalControlSettings` | tekenreeks | `countriesBlockedForMinors` Hiermee geeft u de landen waar de app is geblokkeerd voor minderjarigen.<br>`legalAgeGroupRule` Hiermee geeft u de regel voor meerderjarig die van toepassing op gebruikers van de app. Kan worden ingesteld op `Allow`, `RequireConsentForPrivacyServices`, `RequireConsentForMinors`, `RequireConsentForKids`, of `BlockMinors`.  | <code>{<br>&nbsp;&nbsp;&nbsp;"countriesBlockedForMinors":[],<br>&nbsp;&nbsp;&nbsp;"legalAgeGroupRule":"Allow"<br>} </code> |
| `passwordCredentials` | Het type matrix | Zie de beschrijving voor de `keyCredentials` eigenschap. | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;"customKeyIdentifier":null,<br>&nbsp;&nbsp;&nbsp;"endDate":"2018-10-19T17:59:59.6521653Z",<br>&nbsp;&nbsp;&nbsp;"keyId":"\<guid>",<br>&nbsp;&nbsp;&nbsp;"startDate":"2016-10-19T17:59:59.6521653Z",<br>&nbsp;&nbsp;&nbsp;"value":null<br>&nbsp;&nbsp;&nbsp;}<br>] </code> |
| `preAuthorizedApplications` | Het type matrix | Een lijst met toepassingen en machtigingen die zijn aangevraagd voor impliciete toestemming. Moet een beheerder toestemming voor de toepassing hebt opgegeven. preAuthorizedApplications hoeft niet de gebruiker akkoord gaan met de machtigingen die zijn aangevraagd. Machtigingen die worden vermeld in preAuthorizedApplications vereisen geen toestemming van de gebruiker. Eventuele aanvullende vereiste machtigingen niet wordt vermeld in preAuthorizedApplications is echter toestemming van de gebruiker vereist. | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;"appId": "abcdefg2-000a-1111-a0e5-812ed8dd72e8",<br>&nbsp;&nbsp;&nbsp;&nbsp;"permissionIds": [<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"8748f7db-21fe-4c83-8ab5-53033933c8f1"<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>&nbsp;&nbsp;}<br>]</code> |
| `replyUrlsWithType` | String-matrix | Deze eigenschap meerdere waarden bevat de lijst met geregistreerde redirect_uri waarden waarmee Azure AD worden opgenomen als doel bij het retourneren van tokens. Elke uri-waarde moet een waarde voor het gekoppelde app bevatten. Ondersteunde waarden zijn: `Web`, `InstalledClient`. | <code>"replyUrlsWithType":&nbsp;[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"url":&nbsp;"https://localhost:4400/services/office365/redirectTarget.html",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":&nbsp;"InstalledClient"&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;}<br>]</code> |
| `requiredResourceAccess` | Het type matrix | Met dynamische toestemming `requiredResourceAccess` schijven van de manier waarop beheerders toestemming en de ervaring van de gebruiker toestemming voor gebruikers dat van statische toestemming gebruikmaakt. Dit wordt echter niet de ervaring van de gebruiker toestemming voor de algemene aanvraag station.<br>`resourceAppId` de unieke id voor de resource waarvoor toegang tot de app vereist is. Deze waarde moet gelijk zijn aan de appId gedeclareerd voor de doel-resource-app.<br>`resourceAccess` is een matrix met een lijst met de OAuth 2.0-machtigingsbereiken en app-rollen die de app nodig van de opgegeven resource heeft. Bevat de `id` en `type` waarden van de opgegeven resources. | <code>[<br>&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;"resourceAppId":"00000002-0000-0000-c000-000000000000",<br>&nbsp;&nbsp;&nbsp;&nbsp;"resourceAccess":[<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"id":"311a71cc-e848-46a1-bdf8-97ff7156d8e6",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type":"Scope"<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>&nbsp;&nbsp;}<br>] </code> |
| `samlMetadataUrl` | tekenreeks | De URL naar SAML-metagegevens voor de app. | `https://MyRegisteredAppSAMLMetadata` |
| `signInUrl` | tekenreeks | Hiermee geeft u de URL naar de startpagina van de app. | `https://MyRegisteredApp` |
| `signInAudience` | tekenreeks | Hiermee geeft u op welke microsoft-accounts worden ondersteund voor de huidige toepassing. Ondersteunde waarden zijn:<ul><li>**AzureADMyOrg** -gebruikers met een Microsoft werk- of schoolaccount in mijn organisatie Azure AD-tenant (dat wil zeggen enkele tenant)</li><li>**AzureADMultipleOrgs** -gebruikers met een Microsoft werk- of schoolaccount in elke organisatie Azure AD-tenant (dat wil zeggen met meerdere tenants)</li> <li>**AzureADandPersonalMicrosoftAccount** -gebruikers met een persoonlijk Microsoft-account of een account voor werk of school in elke organisatie Azure AD-tenant</li></ul> | `AzureADandPersonalMicrosoftAccount` |
| `tags` | String-matrix | Aangepaste tekenreeksen die kunnen worden gebruikt voor het categoriseren en identificeren van de toepassing. | <code>[<br>&nbsp;&nbsp;"ProductionApp"<br>]</code> |

## <a name="next-steps"></a>Volgende stappen

* Zie voor meer informatie over de relatie tussen de toepassing en Service-Principal objecten van een app [toepassing en service-principalobjecten in Azure AD](app-objects-and-service-principals.md).
* Zie de [woordenlijst voor ontwikkelaars van Azure AD](developer-glossary.md) voor definities van de basisconcepten voor de ontwikkelaars van Azure Active Directory (AD).

Gebruik de volgende sectie met opmerkingen om feedback die helpt bij het verfijnen en onze inhoud vorm te geven.

<!--article references -->
[AAD-APP-OBJECTS]:app-objects-and-service-principals.md
[AAD-DEVELOPER-GLOSSARY]:developer-glossary.md
[AAD-GROUPS-FOR-AUTHORIZATION]: http://www.dushyantgill.com/blog/2014/12/10/authorization-cloud-applications-using-ad-groups/
[ADD-UPD-RMV-APP]:quickstart-v1-integrate-apps-with-azure-ad.md
[APPLICATION-ENTITY]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity
[APPLICATION-ENTITY-APP-ROLE]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type
[APPLICATION-ENTITY-OAUTH2-PERMISSION]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type
[AZURE-PORTAL]: https://portal.azure.com
[DEV-GUIDE-TO-AUTH-WITH-ARM]: http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/
[GRAPH-API]: active-directory-graph-api.md
[IMPLICIT-GRANT]:v1-oauth2-implicit-grant-flow.md
[INTEGRATING-APPLICATIONS-AAD]: https://azure.microsoft.com/documentation/articles/active-directory-integrating-applications/
[O365-PERM-DETAILS]: https://msdn.microsoft.com/office/office365/HowTo/application-manifest
[O365-SERVICE-DAEMON-APPS]: https://msdn.microsoft.com/office/office365/howto/building-service-apps-in-office-365
[RBAC-CLOUD-APPS-AZUREAD]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/
