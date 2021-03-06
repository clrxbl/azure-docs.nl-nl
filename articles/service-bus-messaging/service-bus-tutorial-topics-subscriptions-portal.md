---
title: 'Zelfstudie: het voorraadassortiment van een detailhandelsbedrijf bijwerken via het publiceren van/abonneren op kanalen en via onderwerpfilters met Azure Portal | Microsoft Docs'
description: In deze zelfstudie leert u hoe u berichten kunt versturen naar en ontvangen van een onderwerp en abonnement, en hoe u filterregels kunt toevoegen en gebruiken met behulp van .NET.
services: service-bus-messaging
author: spelluru
manager: timlt
ms.author: spelluru
ms.date: 09/22/2018
ms.topic: tutorial
ms.service: service-bus-messaging
ms.custom: mvc
ms.openlocfilehash: 357bdcbbee348d8cb89e2d75060a3e7ba05e2c86
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406231"
---
# <a name="tutorial-update-inventory-using-azure-portal-and-topicssubscriptions"></a>Zelfstudie: voorraad bijwerken met Azure Portal en onderwerpen/abonnementen

Microsoft Azure Service Bus is een multitenant-cloudberichtenservice waarmee u gegevens kunt versturen tussen toepassingen en services. Asynchrone bewerkingen bieden flexibele, Brokered Messaging, samen met gestructureerde FIFO-berichtenuitwisseling (first in, first out) en mogelijkheden voor publiceren/abonneren. In deze zelfstudie leert hoe u onderwerpen en abonnementen van Service Bus kunt gebruiken in een scenario met detailhandelvoorraad, waarbij gebruik wordt gemaakt van kanalen voor publiceren/abonneren met Azure Portal en .NET.

In deze zelfstudie leert u het volgende:
> [!div class="checklist"]
> * Een Service Bus-onderwerp en een of meer abonnementen voor dat onderwerp maken met Azure Portal
> * Onderwerpfilters toevoegen met .NET-code
> * Twee berichten met verschillende inhoud maken
> * De berichten verzenden en controleren of ze zijn ontvangen in de verwachte abonnementen
> * Berichten ontvangen van de abonnementen

Een voorbeeld van dit scenario is het bijwerken van het voorraadassortiment voor meerdere winkels. In dit scenario ontvangt elke winkel of reeks winkels berichten op basis waarvan ze hun assortiment kunnen bijwerken. Deze zelfstudie laat zien hoe dit scenario met behulp van abonnementen en filters kan worden geïmplementeerd. Eerst maakt u een onderwerp met drie abonnementen, voegt u een aantal regels en filters toe, en vervolgens verzendt en ontvangt u berichten van het onderwerp en de abonnementen.

![onderwerp](./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png)

Als u nog geen abonnement op Azure hebt, kunt u een [gratis account][] maken voordat u begint.

## <a name="prerequisites"></a>Vereisten

Het volgende moet zijn geïnstalleerd om deze zelfstudie te voltooien:

- [Visual Studio 2017 update 3 (versie 15.3, 26730.01)](http://www.visualstudio.com/vs) of hoger.
- [NET Core SDK](https://www.microsoft.com/net/download/windows), versie 2.0 of later.

## <a name="service-bus-topics-and-subscriptions"></a>Service Bus-onderwerpen en -abonnementen

Elk [abonnement op een onderwerp](service-bus-messaging-overview.md#topics) kan een kopie van elk bericht ontvangen. Onderwerpen zijn qua protocol en semantiek volledig compatibel met Service Bus-wachtrijen. Service Bus-onderwerpen ondersteunen een breed scala aan selectieregels met filtervoorwaarden, met optionele acties waarmee berichteigenschappen worden ingesteld of gewijzigd. Telkens wanneer een regel overeenkomt, wordt er een bericht gegenereerd. Volg deze [koppeling](topic-filters.md) voor meer informatie over regels, filters en acties.

## <a name="sign-in-to-the-azure-portal"></a>Aanmelden bij Azure Portal

Ga eerst naar [Azure Portal][Azure portal] en meld u aan met uw Azure-abonnement. De eerste stap bestaat uit het maken van een Service Bus-naamruimte van het type **Berichten**.

## <a name="create-a-service-bus-namespace"></a>Een Service Bus-naamruimte maken

Een Service Bus-berichtennaamruimte biedt een unieke scoping container, waarnaar wordt verwezen met de [volledig gekwalificeerde domeinnaam][] (FQDN), waarin u een of meer wachtrijen, onderwerpen en abonnementen maakt. In het volgende voorbeeld wordt een Service Bus-berichtennaamruimte in een nieuwe of bestaande [resourcegroep](/azure/azure-resource-manager/resource-group-portal) gemaakt:

1. Klik in het linkernavigatievenster van de portal achtereenvolgens op **+ Een resource maken**, **Bedrijfsintegratie** en **Service Bus**.
2. Voer in het dialoogvenster **Naamruimte maken** een naam in voor de naamruimte. In het systeem wordt onmiddellijk gecontroleerd of de naam beschikbaar is.
3. Nadat u hebt gecontroleerd of de naam van de naamruimte beschikbaar is, kiest u de prijscategorie (Standard of Premium).
4. Kies in het veld **Abonnement** een Azure-abonnement waarin u de naamruimte maakt.
5. Kies in het veld **Resourcegroep** een bestaande resourcegroep waarin de naamruimte zal zijn opgenomen of maak een nieuwe resourcegroep.      
6. Kies in **Locatie** het land of regio waarin uw naamruimte moet worden gehost.
7. Klik op **Create**. Uw naamruimte wordt nu gemaakt en ingeschakeld. U moet wellicht enkele minuten wachten terwijl de resources voor uw account worden ingericht.

  ![naamruimte](./media/service-bus-tutorial-topics-subscriptions-portal/create-namespace.png)

### <a name="obtain-the-management-credentials"></a>De beheerreferenties ophalen

Bij het maken van een nieuwe naamruimte wordt automatisch een eerste SAS-regel (Shared Access Signature) gegenereerd met een bijbehorende primaire en secundaire sleutel die elk volledige controle over alle aspecten van de naamruimte bieden. Volg deze stappen om de eerste regel te kopiëren:

1. Klik op **Alle resources** en klik vervolgens op de zojuist gemaakte naam voor de naamruimte.
2. Klik in het venster van de naamruimte op **Beleid voor gedeelde toegang**.
3. Klik in het scherm **Beleid voor gedeelde toegang** op **RootManageSharedAccessKey**.
4. Klik in het scherm **Beleid: RootManageSharedAccessKey** op de knop **Kopiëren** naast **Primaire verbindingsreeks** om de verbindingsreeks naar het klembord te kopiëren voor later gebruik. Plak deze waarde in Kladblok of een andere tijdelijke locatie.

    ![connection-string][connection-string]
5. Herhaal de vorige stap: het kopiëren en plakken van de waarde voor de **Primaire sleutel** voor een tijdelijke locatie zodat u deze later kunt gebruiken.

## <a name="create-a-topic-and-subscriptions"></a>Een onderwerp en abonnementen maken

Als u een Service Bus-wachtrij wilt maken, moet u de naamruimte opgeven waarin u de wachtrij wilt maken. Het volgende voorbeeld laat zien hoe u in de portal een wachtrij maakt:

1. Klik in het navigatiedeelvenster links in de portal op **Service Bus** (als u geen **Service Bus** ziet, klik dan op **Alle services**).
2. Klik op de naamruimte waarin u het onderwerp wilt maken.
3. Klik in het venster voor de naamruimte op **Onderwerpen** en klik vervolgens in het venster **Onderwerpen** op **+ Onderwerpen**.
4. Voer een waarde in voor **Onderwerpnaam** en hanteer voor de andere waarden de standaardinstellingen.
5. Klik onder aan het venster op **Maken**.
6. Noteer de naam van het onderwerp.
7. Selecteer het onderwerp dat u zojuist hebt gemaakt.
8. Klik op **+ Abonnement**, voer de naam van het abonnement in (**S1**) en laat verder de standaardwaarden staan.
9. Herhaal de vorige stap nog tweemaal om abonnementen met de naam **S2** en **S3** te maken.

## <a name="create-filter-rules-on-subscriptions"></a>Filterregels voor abonnementen maken

Nadat de naamruimte en onderwerp/abonnementen zijn ingericht en u over de benodigde referenties beschikt, bent u klaar om filterregels voor de abonnementen te maken, en kunt u vervolgens berichten verzenden en ontvangen. U kunt de code in [deze GitHub-voorbeeldmap](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/GettingStarted\BasicSendReceiveTutorialwithFilters) controleren.

### <a name="send-and-receive-messages"></a>Berichten verzenden en ontvangen

Ga als volgt te werk om de code uit te voeren:

1. Ga naar een gewone opdrachtprompt of een opdrachtprompt van PowerShell en kloon de [Service Bus GitHub-opslagplaats](https://github.com/Azure/azure-service-bus/) met behulp van de volgende opdracht:

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. Navigeer naar de voorbeeldmap `azure-service-bus\samples\DotNet\GettingStarted\BasicSendReceiveTutorialwithFilters`.

3. Pak de verbindingsreeks erbij die u in de sectie [De beheerreferenties ophalen](#obtain-the-management-credentials) van deze zelfstudie naar Kladblok hebt gekopieerd. U hebt ook de naam nodig van het onderwerp dat u in de vorige sectie hebt gemaakt.

4. Typ de volgende opdracht bij de opdrachtprompt:

   ```shell
   dotnet build
   ```

5. Navigeer naar de map `BasicSendReceiveTutorialwithFilters\bin\Debug\netcoreapp2.0`.

6. Typ de volgende opdracht om het programma uit te voeren. Vervang `myConnectionString` door de waarde die u eerder hebt verkregen en vervang `myTopicName` door de naam van het onderwerp dat u hebt gemaakt:

   ```shell
   dotnet BasicSendReceiveTutorialwithFilters.dll -ConnectionString "myConnectionString" -TopicName "myTopicName"
   ``` 
7. Volg de instructies in de console om eerst filters te maken. Een onderdeel van het maken van filters is het verwijderen van de standaardfilters. Wanneer u PowerShell of CLI gebruikt, hoeft u de standaardfilters niet te verwijderen. Maar als u programmacode gebruikt, moet u ze wel verwijderen. De consoleopdrachten 1 en 3 helpen bij het beheren van de filters voor de abonnementen die u eerder hebt gemaakt:

   - Voer 1: uit om de standaardfilters te verwijderen.
   - Voer 2: uit om uw eigen filters toe te voegen.
   - Voer 3: uit om desgewenst uw eigen filters te verwijderen. Dit betekent niet dat de standaardfilters opnieuw worden gemaakt.

    ![Uitvoer van 2](./media/service-bus-tutorial-topics-subscriptions-portal/create-rules.png)

8. Nadat de filters zijn gemaakt, kunt u berichten verzenden. Druk op 4 en u ziet dat er 10 berichten naar het onderwerp worden verzonden:

    ![Verzonden uitvoer](./media/service-bus-tutorial-topics-subscriptions-portal/send-output.png)

9. Druk op 5 en u ziet dat er berichten binnenkomen. Als u geen 10 berichten hebt ontvangen, drukt u op 'm' om het menu weer te geven en drukt u vervolgens nogmaals op 5.

    ![Ontvangen uitvoer](./media/service-bus-tutorial-topics-subscriptions-portal/receive-output.png)

## <a name="clean-up-resources"></a>Resources opschonen

Wanneer u de naamruimte en de wachtrij niet meer nodig hebt, kunt u deze verwijderen. Selecteer deze resources hiervoor in de portal en klik op **Verwijderen**.

## <a name="understand-the-sample-code"></a>De voorbeeldcode begrijpen

Deze sectie bevat meer informatie over de werking van de voorbeeldcode.

### <a name="get-connection-string-and-topic"></a>Verbindingsreeks en onderwerp ophalen

De code declareert eerst een set variabelen die de uitvoering van de rest van het programma bepalen.

```csharp
string ServiceBusConnectionString;
string TopicName;

static string[] Subscriptions = { "S1", "S2", "S3" };
static IDictionary<string, string[]> SubscriptionFilters = new Dictionary<string, string[]> {
    { "S1", new[] { "StoreId IN('Store1', 'Store2', 'Store3')", "StoreId = 'Store4'"} },
    { "S2", new[] { "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'" } },
    { "S3", new[] { "sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')" } }
};
// You can have only have one action per rule and this sample code supports only one action for the first filter, which is used to create the first rule. 
static IDictionary<string, string> SubscriptionAction = new Dictionary<string, string> {
    { "S1", "" },
    { "S2", "" },
    { "S3", "SET sys.Label = 'SalesEvent'"  }
};
static string[] Store = { "Store1", "Store2", "Store3", "Store4", "Store5", "Store6", "Store7", "Store8", "Store9", "Store10" };
static string SysField = "sys.To";
static string CustomField = "StoreId";
static int NrOfMessagesPerStore = 1; // Send at least 1.
```

De verbindingsreeks en de naam van het onderwerp worden via opdrachtregelparameters doorgegeven, zie hieronder, en worden dan gelezen met de methode `Main()`:

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string TopicName = "";

    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i + 1]}");
            ServiceBusConnectionString = args[i + 1]; // Alternatively enter your connection string here.
        }
        else if (args[i] == "-TopicName")
        {
            Console.WriteLine($"TopicName: {args[i + 1]}");
            TopicName = args[i + 1]; // Alternatively enter your queue name here.
        }
    }

    if (ServiceBusConnectionString != "" && TopicName != "")
    {
        Program P = StartProgram(ServiceBusConnectionString, TopicName);
        P.PresentMenu().GetAwaiter().GetResult();
    }
    else
    {
        Console.WriteLine("Specify -Connectionstring and -TopicName to execute the example.");
        Console.ReadKey();
    }
}
```

### <a name="remove-default-filters"></a>Standaardfilters verwijderen

Wanneer u een abonnement maakt, maakt Service Bus een standaardfilter per abonnement. Dit filter maakt het mogelijk om elk bericht te ontvangen dat naar het onderwerp wordt verzonden. Als u aangepaste filters wilt gebruiken, kunt u het standaardfilter verwijderen, zoals wordt weergegeven in de volgende code:

```csharp
private async Task RemoveDefaultFilters()
{
    Console.WriteLine($"Starting to remove default filters.");

    try
    {
        foreach (var subscription in Subscriptions)
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, subscription);
            await s.RemoveRuleAsync(RuleDescription.DefaultRuleName);
            Console.WriteLine($"Default filter for {subscription} has been removed.");
            await s.CloseAsync();
        }

        Console.WriteLine("All default Rules have been removed.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="create-filters"></a>Filters maken

Met de volgende code worden de aangepaste filters toegevoegd die zijn gedefinieerd in deze zelfstudie:

```csharp
private async Task CreateCustomFilters()
{
    try
    {
        for (int i = 0; i < Subscriptions.Length; i++)
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, Subscriptions[i]);
            string[] filters = SubscriptionFilters[Subscriptions[i]];
            if (filters[0] != "")
            {
                int count = 0;
                foreach (var myFilter in filters)
                {
                    count++;

                    string action = SubscriptionAction[Subscriptions[i]];
                    if (action != "")
                    {
                        await s.AddRuleAsync(new RuleDescription
                        {
                            Filter = new SqlFilter(myFilter),
                            Action = new SqlRuleAction(action),
                            Name = $"MyRule{count}"
                        });
                    }
                    else
                    {
                        await s.AddRuleAsync(new RuleDescription
                        {
                            Filter = new SqlFilter(myFilter),
                            Name = $"MyRule{count}"
                        });
                    }
                }
            }

            Console.WriteLine($"Filters and actions for {Subscriptions[i]} have been created.");
        }

        Console.WriteLine("All filters and actions have been created.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="remove-your-custom-created-filters"></a>Uw aangepaste filters verwijderen

Als u alle filters uit uw abonnement wilt verwijderen, kunt u in de volgende code zien hoe u dat doet:

```csharp
private async Task CleanUpCustomFilters()
{
    foreach (var subscription in Subscriptions)
    {
        try
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, subscription);
            IEnumerable<RuleDescription> rules = await s.GetRulesAsync();
            foreach (RuleDescription r in rules)
            {
                await s.RemoveRuleAsync(r.Name);
                Console.WriteLine($"Rule {r.Name} has been removed.");
            }
            await s.CloseAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    Console.WriteLine("All default filters have been removed.\n");

    await PresentMenu();
}
```

### <a name="send-messages"></a>Berichten verzenden

Het verzenden van berichten naar een onderwerp is vergelijkbaar met het verzenden van berichten naar een wachtrij. Dit voorbeeld laat zien hoe u berichten kunt verzenden met behulp van een takenlijst en asynchrone verwerking:

```csharp
public async Task SendMessages()
{
    try
    {
        TopicClient tc = new TopicClient(ServiceBusConnectionString, TopicName);

        var taskList = new List<Task>();
        for (int i = 0; i < Store.Length; i++)
        {
            taskList.Add(SendItems(tc, Store[i]));
        }

        await Task.WhenAll(taskList);
        await tc.CloseAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    Console.WriteLine("\nAll messages sent.\n");
}

private async Task SendItems(TopicClient tc, string store)
{
    for (int i = 0; i < NrOfMessagesPerStore; i++)
    {
        Random r = new Random();
        Item item = new Item(r.Next(5), r.Next(5), r.Next(5));

        // Note the extension class which is serializing an deserializing messages
        Message message = item.AsMessage();
        message.To = store;
        message.UserProperties.Add("StoreId", store);
        message.UserProperties.Add("Price", item.getPrice().ToString());
        message.UserProperties.Add("Color", item.getColor());
        message.UserProperties.Add("Category", item.getItemCategory());

        await tc.SendAsync(message);
        Console.WriteLine($"Sent item to Store {store}. Price={item.getPrice()}, Color={item.getColor()}, Category={item.getItemCategory()}"); ;
    }
}
```

### <a name="receive-messages"></a>Berichten ontvangen

Berichten worden ook ontvangen via een takenlijst en in de code wordt batchverwerking toegepast. U kunt verzenden en ontvangen met behulp van batchverwerking, maar in dit voorbeeld wordt alleen ontvangen gedemonstreerd. In werkelijkheid zou u de lus niet verlaten, maar zou u deze in stand houden en een hogere waarde voor TimeSpan opgeven, zoals één minuut. De Receive-aanroep naar de broker blijft gedurende deze tijd openstaan en als er berichten binnenkomen, worden deze onmiddellijk geretourneerd en wordt er een nieuwe Receive-aanroep uitgegeven. Dit concept wordt *long polling* genoemd. Het toepassen van de Receive-pump die u kunt zien in de [snelstartgids](service-bus-quickstart-portal.md), en in enkele andere voorbeelden in de opslagplaats, is een meer gangbare optie.

```csharp
public async Task Receive()
{
    var taskList = new List<Task>();
    for (var i = 0; i < Subscriptions.Length; i++)
    {
        taskList.Add(this.ReceiveMessages(Subscriptions[i]));
    }

    await Task.WhenAll(taskList);
}

private async Task ReceiveMessages(string subscription)
{
    var entityPath = EntityNameHelper.FormatSubscriptionPath(TopicName, subscription);
    var receiver = new MessageReceiver(ServiceBusConnectionString, entityPath, ReceiveMode.PeekLock, RetryPolicy.Default, 100);

    while (true)
    {
        try
        {
            IList<Message> messages = await receiver.ReceiveAsync(10, TimeSpan.FromSeconds(2));
            if (messages.Any())
            {
                foreach (var message in messages)
                {
                    lock (Console.Out)
                    {
                        Item item = message.As<Item>();
                        IDictionary<string, object> myUserProperties = message.UserProperties;
                        Console.WriteLine($"StoreId={myUserProperties["StoreId"]}");

                        if (message.Label != null)
                        {
                            Console.WriteLine($"Label={message.Label}");
                        }

                        Console.WriteLine(
                            $"Item data: Price={item.getPrice()}, Color={item.getColor()}, Category={item.getItemCategory()}");
                    }

                    await receiver.CompleteAsync(message.SystemProperties.LockToken);
                }
            }
            else
            {
                break;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }

    await receiver.CloseAsync();
}
```

## <a name="next-steps"></a>Volgende stappen

In deze zelfstudie hebt u resources ingericht met behulp van Azure Portal en vervolgens berichten verzonden en ontvangen van een Service Bus-onderwerp en de daarbij behorende abonnementen. U hebt geleerd hoe u:

> [!div class="checklist"]
> * Een Service Bus-onderwerp en een of meer abonnementen voor dat onderwerp maken met Azure Portal
> * Onderwerpfilters toevoegen met .NET-code
> * Twee berichten met verschillende inhoud maken
> * De berichten verzenden en controleren of ze zijn ontvangen in de verwachte abonnementen
> * Berichten ontvangen van de abonnementen

Meer voorbeelden voor het verzenden en ontvangen van berichten vindt u in [de Service Bus-voorbeelden op GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted).

Ga naar de volgende zelfstudie voor meer informatie over het gebruik van de mogelijkheden voor publiceren/abonneren van Service Bus.

> [!div class="nextstepaction"]
> [Voorraad bijwerken met behulp van PowerShell en onderwerpen/abonnementen](service-bus-tutorial-topics-subscriptions-powershell.md)

[gratis account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[volledig gekwalificeerde domeinnaam]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Azure portal]: https://portal.azure.com/

[connection-string]: ./media/service-bus-tutorial-topics-subscriptions-portal/connection-string.png
[service-bus-flow]: ./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png