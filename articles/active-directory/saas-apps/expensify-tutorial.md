---
title: 'Zelfstudie: Azure Active Directory-integratie met Expensify | Microsoft Docs'
description: Informatie over het configureren van eenmalige aanmelding tussen Azure Active Directory en Expensify.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 1e761484-7a2f-4321-91f4-6d5d0b69344e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/02/2018
ms.author: jeedes
ms.openlocfilehash: c9ee0af3cbf2c1aa7b24d2f4cf5fba9d664bc087
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/03/2018
ms.locfileid: "48248037"
---
# <a name="tutorial-azure-active-directory-integration-with-expensify"></a>Zelfstudie: Azure Active Directory-integratie met Expensify

In deze zelfstudie leert u hoe u Expensify integreren met Azure Active Directory (Azure AD).

Expensify integreren met Azure AD biedt u de volgende voordelen:

- U kunt beheren in Azure AD die toegang tot Expensify heeft.
- U kunt uw gebruikers automatisch ophalen aangemeld bij Expensify (Single Sign-On) met hun Azure AD-accounts inschakelen.
- U kunt uw accounts in één centrale locatie - Azure portal beheren.

Als u wilt graag meer informatie over de integratie van de SaaS-app met Azure AD, Zie [wat is toegang tot toepassingen en eenmalige aanmelding met Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Vereisten

Voor het configureren van Azure AD-integratie met Expensify, moet u de volgende items:

- Een Azure AD-abonnement
- Een Expensify eenmalige aanmelding ingeschakeld abonnement

> [!NOTE]
> Als u wilt testen van de stappen in deze zelfstudie, raden we niet met behulp van een productie-omgeving.

Als u wilt testen van de stappen in deze zelfstudie, moet u deze aanbevelingen volgen:

- Gebruik uw productie-omgeving, niet als dat nodig is.
- Als u geen een proefversie Azure AD-omgeving hebt, kunt u [een proefversie van één maand krijgen](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Scenariobeschrijving
In deze zelfstudie test u de Azure AD eenmalige aanmelding in een testomgeving. Het scenario in deze zelfstudie bestaat uit twee belangrijkste bouwstenen:

1. Expensify uit de galerie toe te voegen
1. Configureren en testen van Azure AD eenmalige aanmelding

## <a name="adding-expensify-from-the-gallery"></a>Expensify uit de galerie toe te voegen

Voor het configureren van de integratie van Expensify in Azure AD, moet u Expensify uit de galerie toevoegen aan uw lijst met beheerde SaaS-apps.

**Als u wilt toevoegen Expensify uit de galerie, moet u de volgende stappen uitvoeren:**

1. In de **[Azure-portal](https://portal.azure.com)**, klik in het navigatievenster aan de linkerkant op **Azure Active Directory** pictogram. 

    ![image](./media/expensify-tutorial/selectazuread.png)

2. Navigeer naar **bedrijfstoepassingen**. Ga vervolgens naar **alle toepassingen**.

    ![image](./media/expensify-tutorial/a_select_app.png)
    
3. Nieuwe toepassing toevoegen, klikt u op **nieuwe toepassing** knop boven aan het dialoogvenster.

    ![image](./media/expensify-tutorial/a_new_app.png)

4. Typ in het zoekvak **Expensify**, selecteer **Expensify** van resultaat deelvenster klik vervolgens op **toevoegen** om toe te voegen van de toepassing.

     ![image](./media/expensify-tutorial/a_add_app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configureren en Azure AD eenmalige aanmelding testen

In deze sectie maakt u configureert en test Azure AD eenmalige aanmelding met Expensify op basis van een testgebruiker 'Julia steen' genoemd.

Voor eenmalige aanmelding om te werken, moet Azure AD om te weten wat de gebruiker equivalent in Expensify is aan een gebruiker in Azure AD. Met andere woorden, moet een koppeling relatie tussen een Azure AD-gebruiker en de gerelateerde gebruiker in Expensify tot stand worden gebracht.

In Expensify, wijs de waarde van de **gebruikersnaam** in Azure AD als de waarde van de **gebruikersnaam** de relatie van de koppeling tot stand brengen.

Om te configureren en testen van Azure AD eenmalige aanmelding met Expensify, moet u de volgende bouwstenen voltooien:

1. **[Azure AD eenmalige aanmelding configureren](#configure-azure-ad-single-sign-on)**  : als u wilt dat uw gebruikers kunnen deze functie gebruiken.
1. **[Maak een Azure AD-testgebruiker](#create-an-azure-ad-test-user)**  - voor het testen van Azure AD eenmalige aanmelding met Britta Simon.
1. **[Maak een testgebruiker Expensify](#create-an-expensify-test-user)**  : als u wilt een equivalent van Britta Simon in Expensify die is gekoppeld aan de Azure AD-weergave van de gebruiker hebben.
1. **[Toewijzen van de Azure AD-testgebruiker](#assign-the-azure-ad-test-user)**  - Britta Simon gebruik van Azure AD eenmalige aanmelding inschakelen.
1. **[Eenmalige aanmelding testen](#test-single-sign-on)**  : als u wilt controleren of de configuratie werkt.

### <a name="configure-azure-ad-single-sign-on"></a>Azure AD eenmalige aanmelding configureren

In deze sectie maakt u schakelt Azure AD eenmalige aanmelding in de Azure-portal en configureren van eenmalige aanmelding in uw toepassing Expensify.

**Voor het configureren van Azure AD eenmalige aanmelding met Expensify, moet u de volgende stappen uitvoeren:**

1. In de [Azure-portal](https://portal.azure.com/)op de **Expensify** toepassing integratie weergeeft, schakelt **eenmalige aanmelding**.

    ![image](./media/expensify-tutorial/b1_b2_select_sso.png)

2. Klik op **modus voor één wijziging aanmelding** boven op het scherm selecteren de **SAML** modus.

      ![image](./media/expensify-tutorial/b1_b2_saml_ssso.png)

3. Op de **selecteert u een methode voor eenmalige aanmelding** dialoogvenster, klikt u op **Selecteer** voor **SAML** modus voor eenmalige aanmelding inschakelen.

    ![image](./media/expensify-tutorial/b1_b2_saml_sso.png)

4. Op de **instellen van eenmalige aanmelding met SAML** pagina, klikt u op **bewerken** te openen **SAML-basisconfiguratie** dialoogvenster.

    ![image](./media/expensify-tutorial/b1-domains_and_urlsedit.png)

5. Op de **SAML-basisconfiguratie** sectie, voert u de volgende stappen uit:

    a. In de **aanmeldings-URL** in het tekstvak, een URL als: `https://www.expensify.com/authentication/saml/login`

    b. In de **id** tekstvak typt u een URL met behulp van het volgende patroon: `https://www.<companyname>.expensify.com`

    ![image](./media/expensify-tutorial/b1-domains_and_urls.png)

    > [!NOTE] 
    > Vervang de <companyname> sectie van de identificatie-URL met het domein van uw bedrijf. Zie het voorbeeld van `https://contoso.expensify.com` hierboven. In Expensify, dit is de naam van uw domein zoals is gedaan in **instellingen > domein besturingselement**.

    ![Expensify domeingegevens](./media/expensify-tutorial/tutorial_expensify_domain.png)

6. Op de **instellen van eenmalige aanmelding met SAML** pagina, in de **SAML-handtekeningcertificaat** sectie, klikt u op **downloaden** voor het downloaden van het juiste certificaat volgens uw vereiste en sla deze op uw computer.

    ![image](./media/expensify-tutorial/certificatebase64.png)

7. Als u wilt inschakelen van eenmalige aanmelding in Expensify, moet u eerst om in te schakelen **domein besturingselement** in de toepassing. U kunt het domein inschakelen in de toepassing via de hier vermelde stappen [hier](http://help.expensify.com/domain-control). Voor aanvullende ondersteuning, werken met [Expensify Client ondersteuningsteam](mailto:help@expensify.com). Zodra u hebt de domein-beheer is ingeschakeld, als volgt te werk:
   
    ![Eenmalige aanmelding configureren](./media/expensify-tutorial/tutorial_expensify_51.png)
    
    a. Meld u aan bij uw toepassing Expensify.
    
    b. Klik in het linkerdeelvenster op **instellingen** en navigeer naar **SAML**.
    
    c. Schakelen tussen de **SAML-aanmelding** als de optie **ingeschakeld**.
    
    d. Open het gedownloade Federatiemetagegevens van Azure AD in Kladblok, Kopieer de inhoud en plak deze in de **Identity Provider metagegevens** tekstvak.

### <a name="create-an-azure-ad-test-user"></a>Maak een testgebruiker Azure AD 

Het doel van deze sectie is het maken van een testgebruiker in Azure portal Britta Simon genoemd.

1. Selecteer in de Azure portal, in het linkerdeelvenster **Azure Active Directory**, selecteer **gebruikers**, en selecteer vervolgens **alle gebruikers**.

    ![image](./media/expensify-tutorial/d_users_and_groups.png)

2. Selecteer **nieuwe gebruiker** aan de bovenkant van het scherm.

    ![image](./media/expensify-tutorial/d_adduser.png)

3. In de eigenschappen van de gebruiker de volgende stappen uitvoeren.

    ![image](./media/expensify-tutorial/d_userproperties.png)

    a. In de **naam** veld **BrittaSimon**.
  
    b. In de **gebruikersnaam** veldtype **brittasimon@yourcompanydomain.extension**  
    Bijvoorbeeld: BrittaSimon@contoso.com

    c. Selecteer **eigenschappen**, selecteer de **Show wachtwoord** selectievakje en noteer de waarde die wordt weergegeven in het wachtwoord.

    d. Selecteer **Maken**.
 
### <a name="create-an-expensify-test-user"></a>Maak een testgebruiker Expensify

In deze sectie maakt u een gebruiker met de naam van Britta Simon in Expensify. Werken met [Expensify Client ondersteuningsteam](mailto:help@expensify.com) om toe te voegen de gebruikers in het Expensify-platform.

### <a name="assign-the-azure-ad-test-user"></a>De Azure AD-testgebruiker toewijzen

In deze sectie maakt inschakelen u Britta Simon gebruiken Azure eenmalige aanmelding door toegang te verlenen aan Expensify.

1. Selecteer in de Azure portal, **bedrijfstoepassingen**, selecteer **alle toepassingen**.

    ![image](./media/expensify-tutorial/d_all_applications.png)

2. Selecteer in de lijst met toepassingen, **Expensify**.

    ![image](./media/expensify-tutorial/d_all_proapplications.png)

3. Selecteer in het menu aan de linkerkant, **gebruikers en groepen**.

    ![image](./media/expensify-tutorial/d_leftpaneusers.png)

4. Selecteer de **toevoegen** knop en selecteer vervolgens **gebruikers en groepen** in de **toevoegen toewijzing** dialoogvenster.

    ![image](./media/expensify-tutorial/d_assign_user.png)

4. In de **gebruikers en groepen** dialoogvenster Selecteer **Britta Simon** in de lijst met gebruikers, en klik op de **Selecteer** knop aan de onderkant van het scherm.

5. In de **toevoegen toewijzing** dialoogvenster Selecteer de **toewijzen** knop.
    
### <a name="test-single-sign-on"></a>Eenmalige aanmelding testen

In deze sectie maakt testen u uw Azure AD eenmalige aanmelding configuratie met behulp van het toegangsvenster.

Wanneer u op de tegel Expensify in het toegangsvenster, u moet u automatisch aangemeld bij uw toepassing Expensify.
Zie voor meer informatie over het toegangsvenster, [Inleiding tot het toegangsvenster](../user-help/active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Aanvullende resources

* [Lijst met zelfstudies over het integreren van SaaS-Apps met Azure Active Directory](tutorial-list.md)
* [What is application access and single sign-on with Azure Active Directory?](../manage-apps/what-is-single-sign-on.md) (Wat houden toegang tot toepassingen en eenmalige aanmelding met Azure Active Directory in?)




