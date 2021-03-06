---
title: Een door het Windows-VM-systeem toegewezen beheerde identiteit gebruiken voor toegang tot Azure SQL
description: Een zelfstudie die u helpt bij het doorlopen van het proces voor het gebruiken van een door het Windows-VM-systeem toegewezen beheerde identiteit om toegang te krijgen tot Azure SQL.
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: bryanla
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: daveba
ms.openlocfilehash: 57f9def09f498c3fc644cbee979d5ae552f2206c
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/19/2018
ms.locfileid: "46369391"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-sql"></a>Zelfstudie: een door het Windows-VM-systeem toegewezen beheerde identiteit gebruiken voor toegang tot Azure SQL

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Deze zelfstudie laat zien hoe u toegang krijgt tot een Azure SQL-server met een door het systeem toegewezen identiteit voor een virtuele Windows-machine (VM). Managed Service Identity's worden automatisch beheerd in Azure en stellen u in staat om te verifiëren bij services die Azure AD-verificatie ondersteunen, zonder referenties in code te hoeven invoegen. In deze zelfstudie leert u procedures om het volgende te doen:

> [!div class="checklist"]
> * Uw virtuele machine toegang verlenen tot een Azure SQL-server
> * Een groep in Azure AD maken en de door het systeem toegewezen beheerde identiteit van de VM lid van de groep maken
> * Azure AD-verificatie inschakelen voor de SQL-server
> * Een ingesloten gebruiker maken in de database die staat voor de Azure AD-groep
> * Een toegangstoken ophalen met de identiteit van de virtuele machine en daarmee een query uitvoeren op een Azure SQL-server

## <a name="prerequisites"></a>Vereisten

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [Aanmelden bij de Azure-portal](https://portal.azure.com)

- [Een virtuele Windows-machine maken](/azure/virtual-machines/windows/quick-create-portal)

- [Door het systeem toegewezen beheerde identiteit op uw virtuele machine inschakelen](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="grant-your-vm-access-to-a-database-in-an-azure-sql-server"></a>Uw virtuele machine toegang verlenen tot een database op een Azure SQL-server

U kunt nu uw virtuele machine toegang verlenen tot een database op een Azure SQL-server.  Voor deze stap kunt u een bestaande SQL-server gebruiken, of een nieuwe maken.  Voor het maken van een nieuwe server en database met behulp van Azure Portal, volgt u deze [Azure SQL-snelstart](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal). Er zijn ook snelstarts in de [documentatie over Azure SQL](https://docs.microsoft.com/azure/sql-database/) voor het gebruik van Azure CLI en Azure Powershell.

U moet drie stappen uitvoeren om uw virtuele machine toegang te verlenen tot een database:
1.  Maak een groep in Azure AD en maak de door het systeem toegewezen beheerde identiteit van de VM lid van de groep.
2.  Schakel Azure AD-verificatie in voor de SQL-server.
3.  Maak een **ingesloten gebruiker** in de database die staat voor de Azure AD-groep.

> [!NOTE]
> Doorgaans maakt u een ingesloten gebruiker die direct is gekoppeld aan de door het systeem toegewezen beheerde identiteit van de VM.  Momenteel is het in Azure SQL niet mogelijk om de Azure AD-service-principal die de door het systeem toegewezen beheerde identiteit van de VM vertegenwoordigt, te koppelen aan een ingesloten gebruiker.  Als ondersteunde oplossing maakt u de door het systeem toegewezen beheerde identiteit van de VM lid van een Azure AD-groep en maakt u vervolgens een ingesloten gebruiker in de database die staat voor de groep.


## <a name="create-a-group-in-azure-ad-and-make-the-vms-system-assigned-managed-identity-a-member-of-the-group"></a>Een groep in Azure AD maken en de door het systeem toegewezen beheerde identiteit van de VM lid van de groep maken

U kunt een bestaande Azure AD-groep gebruiken of een nieuwe groep maken met behulp van Azure AD PowerShell.  

Installeer eerst de module [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2). Meld u vervolgens aan met behulp van `Connect-AzureAD`, en voer de volgende opdracht uit om de groep te maken en op te slaan in een variabele:

```powershell
$Group = New-AzureADGroup -DisplayName "VM managed identity access to SQL" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
```

De uitvoer ziet er ongeveer als volgt uit, waarbij ook de waarde van de variabele wordt onderzocht:

```powershell
$Group = New-AzureADGroup -DisplayName "VM managed identity access to SQL" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
$Group
ObjectId                             DisplayName          Description
--------                             -----------          -----------
6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 VM managed identity access to SQL
```

Voeg vervolgens de door het systeem toegewezen beheerde identiteit van de VM aan de groep toe.  Hiervoor hebt u de **object-id** van de door het systeem toegewezen beheerde identiteit nodig. U kunt deze ophalen met behulp van Azure PowerShell.  Download eerst [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps). Meld u vervolgens aan met behulp van `Connect-AzureRmAccount`, en voer de volgende opdrachten uit om het volgende te doen:
- Controleren of de sessiecontext is ingesteld op het gewenste Azure-abonnement, als u meerdere abonnementen hebt.
- Weergeven van de beschikbare resources in uw Azure-abonnement om te verifiëren of de naam van de resourcegroep en de naam van de virtuele machine juist zijn ingesteld.
- De eigenschappen van de door het systeem toegewezen beheerde identiteit van de VM ophalen met behulp van de juiste waarden voor `<RESOURCE-GROUP>` en `<VM-NAME>`.

```powershell
Set-AzureRMContext -subscription "bdc79274-6bb9-48a8-bfd8-00c140fxxxx"
Get-AzureRmResource
$VM = Get-AzureRmVm -ResourceGroup <RESOURCE-GROUP> -Name <VM-NAME>
```

De uitvoer ziet er ongeveer als volgt uit, waarbij ook de service-principal-object-id van de door het systeem toegewezen beheerde identiteit van de VM wordt onderzocht:
```powershell
$VM = Get-AzureRmVm -ResourceGroup DevTestGroup -Name DevTestWinVM
$VM.Identity.PrincipalId
b83305de-f496-49ca-9427-e77512f6cc64
```

Voeg nu de door het systeem toegewezen beheerde identiteit van de VM aan de groep toe.  U kunt alleen een service-principal aan een groep toevoegen met behulp van Azure AD PowerShell.  Voer deze opdracht uit:
```powershell
Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId $VM.Identity.PrincipalId
```

Als u daarna ook het groepslidmaatschap onderzoekt, ziet de uitvoer er ongeveer als volgt uit:

```powershell
Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId $VM.Identity.PrincipalId
Get-AzureAdGroupMember -ObjectId $Group.ObjectId

ObjectId                             AppId                                DisplayName
--------                             -----                                -----------
b83305de-f496-49ca-9427-e77512f6cc64 0b67a6d6-6090-4ab4-b423-d6edda8e5d9f DevTestWinVM
```

## <a name="enable-azure-ad-authentication-for-the-sql-server"></a>Azure AD-verificatie inschakelen voor de SQL-server

U hebt de groep gemaakt en u hebt de door het systeem toegewezen beheerde identiteit van de VM toegevoegd als lid van de groep. Nu kunt u [Azure AD-verificatie configureren voor de SQL-server](/azure/sql-database/sql-database-aad-authentication-configure#provision-an-azure-active-directory-administrator-for-your-azure-sql-server) met behulp van de volgende stappen:

1.  Selecteer **SQL-servers** in het linkernavigatievenster van Azure Portal.
2.  Klik op de SQL-server waarvoor Azure AD-verificatie moet worden ingeschakeld.
3.  Klik in de sectie **Instellingen** van de blade op **Active Directory-beheerder**.
4.  Klik in de opdrachtbalk op **Beheerder instellen**.
5.  Selecteer een Azure AD-gebruikersaccount dat beheerder van de server moet worden gemaakt en klik op **Selecteren**.
6.  Klik in de opdrachtbalk op **Opslaan**.

## <a name="create-a-contained-user-in-the-database-that-represents-the-azure-ad-group"></a>Een ingesloten gebruiker maken in de database die staat voor de Azure AD-groep

Voor de volgende stap hebt u [Microsoft SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) nodig. Voordat u begint, kan het ook handig zijn de volgende artikelen te lezen voor meer achtergrondinformatie over Azure AD-integratie:

- [Universele verificatie met SQL Database en SQL Data Warehouse (SSMS-ondersteuning voor MFA)](/azure/sql-database/sql-database-ssms-mfa-authentication)
- [Azure Active Directory-verificatie configureren en beheren met SQL Database of SQL Data Warehouse](/azure/sql-database/sql-database-aad-authentication-configure)

1.  Start SQL Server Management Studio.
2.  Voer in het dialoogvenster **Verbinding maken met server** de naam van de SQL-server in het veld **Servernaam** in.
3.  Selecteer in het veld **Verificatie** **Active Directory - Universeel met ondersteuning voor MFA**.
4.  Voer in het veld **Gebruikersnaam** de naam in van het Azure AD-account dat u hebt ingesteld als de beheerder van de server, bijvoorbeeld helen@woodgroveonline.com
5.  Klik op **Opties**.
6.  Voer in het veld **Verbinding maken met database** de naam in van de niet-systeemdatabase die u wilt configureren.
7.  Klik op **Verbinden**.  Voltooi het aanmeldingsproces.
8.  Vouw in **Objectverkenner** de map **Databases** uit.
9.  Klik met de rechtermuisknop op een gebruikersdatabase en klik op **Nieuwe query**.
10.  Voer in het queryvenster de volgende regel in en klik op **Uitvoeren** in de werkbalk:
    
     ```
     CREATE USER [VM managed identity access to SQL] FROM EXTERNAL PROVIDER
     ```
    
     De opdracht voor het maken van de ingesloten gebruiker voor de groep wordt uitgevoerd.
11.  Wis het queryvenster, voer de volgende regel in en klik op **Uitvoeren** in de werkbalk:
     
     ```
     ALTER ROLE db_datareader ADD MEMBER [VM managed identity access to SQL]
     ```

     De opdracht wordt uitgevoerd en de ingesloten gebruiker heeft nu leestoegang tot de gehele database.

Code die wordt uitgevoerd op de VM kan nu een token verkrijgen via de door het systeem toegewezen beheerde identiteit en het token gebruiken voor verificatie bij de SQL-server.

## <a name="get-an-access-token-using-the-vms-system-assigned-managed-identity-and-use-it-to-call-azure-sql"></a>Een toegangstoken ophalen met behulp van de door het systeem toegewezen beheerde identiteit van de VM en dit gebruiken om Azure SQL aan te roepen 

Azure SQL biedt systeemeigen ondersteuning voor Azure AD-verificatie, zodat toegangstokens die zijn verkregen met behulp van beheerde identiteiten voor Azure-resources direct kunnen worden geaccepteerd.  U gebruikt de toegangsmethode met het **toegangstoken** voor het maken van een verbinding met SQL.  Dit maakt deel uit van de integratie van Azure SQL met Azure AD en wijkt af van het opgeven van referenties in de verbindingsreeks.

Hier volgt een voorbeeld van .NET-code voor het openen van een verbinding met SQL met behulp van een toegangstoken.  Deze code moet worden uitgevoerd op de virtuele machine om toegang te krijgen tot het eindpunt van de door het systeem toegewezen beheerde identiteit van de virtuele machine.  **.NET framework 4.6** of hoger is vereist voor het gebruik van de toegangsmethode met een toegangstoken.  Vervang AZURE-SQL-SERVERNAME en DATABASE door de benodigde waarden.  De resource-id voor Azure SQL is https://database.windows.net/.

```csharp
using System.Net;
using System.IO;
using System.Data.SqlClient;
using System.Web.Script.Serialization;

//
// Get an access token for SQL.
//
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://database.windows.net/");
request.Headers["Metadata"] = "true";
request.Method = "GET";
string accessToken = null;

try
{
    // Call managed identities for Azure resources endpoint.
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader and extract access token.
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

//
// Open a connection to the SQL server using the access token.
//
if (accessToken != null) {
    string connectionString = "Data Source=<AZURE-SQL-SERVERNAME>; Initial Catalog=<DATABASE>;";
    SqlConnection conn = new SqlConnection(connectionString);
    conn.AccessToken = accessToken;
    conn.Open();
}
```

Met PowerShell kunt u snel de end-to-end-instellingen testen zonder een app te hoeven schrijven en implementeren op de virtuele machine.

1.  Navigeer in Azure Portal naar **Virtuele machines**, ga naar uw virtuele Windows-machine en klik op de pagina **Overzicht** op **Verbinden**. 
2.  Voer uw referenties (**gebruikersnaam** en **wachtwoord**) in die u hebt toegevoegd bij het maken van de virtuele Windows-machine. 
3.  Nu u een **Verbinding met extern bureaublad** met de virtuele machine hebt gemaakt, opent u **PowerShell** in de externe sessie. 
4.  Verstuur met de cmdlet `Invoke-WebRequest` van Powershell een aanvraag naar het lokale eindpunt van de beheerde identiteit om een toegangstoken voor Azure SQL op te halen.

    ```powershell
       $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"}
    ```
    
    Converteer de reactie van een JSON-object naar een PowerShell-object. 
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```

    Extraheer het toegangstoken uit de reactie.
    
    ```powershell
    $AccessToken = $content.access_token
    ```

5.  Open een verbinding met de SQL-server. Vervang de waarden voor AZURE-SQL-SERVERNAME en DATABASE.
    
    ```powershell
    $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
    $SqlConnection.ConnectionString = "Data Source = <AZURE-SQL-SERVERNAME>; Initial Catalog = <DATABASE>"
    $SqlConnection.AccessToken = $AccessToken
    $SqlConnection.Open()
    ```

    Maak vervolgens een query en verzend de query naar de server.  Vergeet niet de waarde voor TABLE te vervangen.

    ```powershell
    $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
    $SqlCmd.CommandText = "SELECT * from <TABLE>;"
    $SqlCmd.Connection = $SqlConnection
    $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
    $SqlAdapter.SelectCommand = $SqlCmd
    $DataSet = New-Object System.Data.DataSet
    $SqlAdapter.Fill($DataSet)
    ```

Bekijk de waarde van `$DataSet.Tables[0]` om de resultaten van de query weer te geven.

## <a name="next-steps"></a>Volgende stappen

In deze zelfstudie hebt u geleerd om met een door het systeem toegewezen beheerde identiteit toegang te krijgen tot Azure SQL Server.  Zie voor meer informatie over Azure SQL Server:

> [!div class="nextstepaction"]
>[Azure SQL Database Service](/azure/sql-database/sql-database-technical-overview)
