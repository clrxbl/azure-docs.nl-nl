---
title: Azure Functions testen | Microsoft Docs
description: Uw Azure functions testen met behulp van Postman cURL en Node.js.
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
keywords: Azure functions, functies, gebeurtenisverwerking, webhooks, dynamisch berekenen, architectuur zonder server, testen
ms.assetid: c00f3082-30d2-46b3-96ea-34faf2f15f77
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 02/02/2017
ms.author: glenga
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1aa1c3a3c0dd4985662b5ceb4acd7250cf2d0186
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 09/07/2018
ms.locfileid: "44093223"
---
# <a name="strategies-for-testing-your-code-in-azure-functions"></a>Strategieën voor het testen van uw code in Azure Functions

In dit onderwerp ziet u de verschillende manieren voor het testen van functies, inclusief het gebruik van de volgende algemene methoden:

+ HTTP-gebaseerde hulpprogramma's, zoals cURL, Postman en zelfs een webbrowser voor web-gebaseerde triggers
+ Azure Storage Explorer om te testen op basis van een Azure Storage-triggers
+ Tabblad in de portal voor Azure Functions testen
+ Timer geactiveerde functie
+ Testen van de toepassing of elk framework

Alle methoden voor deze tests gebruiken een HTTP-triggerfunctie die invoer via een tekenreeksparameter of hoofdtekst van de aanvraag accepteert. U deze functie maken met de Azure-portal in de eerste sectie.

## <a name="create-a-simple-function-for-testing-using-the-azure-portal"></a>Maken van een eenvoudige functie voor het testen met behulp van de Azure portal
Voor de meeste van deze zelfstudie gebruiken we een enigszins gewijzigde versie van de sjabloon van de HttpTrigger JavaScript-functie die beschikbaar is wanneer u een functie maken. Als u hulp bij het maken van een functie, bekijkt u deze [zelfstudie](functions-create-first-azure-function.md). Kies de **HttpTrigger - JavaScript** sjabloon bij het maken van de testfunctie in de [Azure Portal].

De functie standaardsjabloon is in feite een 'Hallo wereld'-functie die de naam van de aanvraag hoofdtekst of de query queryreeks-parameter, een echo terug `name=<your name>`.  De code ook dat u kunt de naam en een adres opgeven als JSON-inhoud in de hoofdtekst van de aanvraag wordt bijgewerkt. De functie kan vervolgens deze terug naar de client wanneer deze beschikbaar.   

De functie met de volgende code, die we gebruiken voor het testen van update:

```javascript
module.exports = function (context, req) {
    context.log("HTTP trigger function processed a request. RequestUri=%s", req.originalUrl);
    context.log("Request Headers = " + JSON.stringify(req.headers));
    var res;

    if (req.query.name || (req.body && req.body.name)) {
        if (typeof req.query.name != "undefined") {
            context.log("Name was provided as a query string param...");
            res = ProcessNewUserInformation(context, req.query.name);
        }
        else {
            context.log("Processing user info from request body...");
            res = ProcessNewUserInformation(context, req.body.name, req.body.address);
        }
    }
    else {
        res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done(null, res);
};
function ProcessNewUserInformation(context, name, address) {
    context.log("Processing user information...");
    context.log("name = " + name);
    var echoString = "Hello " + name;
    var res;

    if (typeof address != "undefined") {
        echoString += "\n" + "The address you provided is " + address;
        context.log("address = " + address);
    }
    res = {
        // status: 200, /* Defaults to 200 */
        body: echoString
    };
    return res;
}
```

## <a name="test-a-function-with-tools"></a>Een functie met hulpprogramma's testen
Er zijn verschillende hulpmiddelen die u gebruiken kunt voor het activeren van uw functies voor het testen van buiten de Azure-portal. Het gaat hierbij om HTTP-hulpprogramma's (zowel op basis van een gebruikersinterface en opdracht regel), toegang tot hulpprogramma's voor Azure Storage en zelfs een eenvoudige webbrowser testen.

### <a name="test-with-a-browser"></a>Testen met een browser
De webbrowser is een eenvoudige manier om triggerfuncties via HTTP. De tekenreeks parameters kunt u een browser voor GET-aanvragen waarvoor de nettolading van een instantie niet vereist, en dat gebruik alleen een query.

De functie te testen we eerder hebt gedefinieerd, Kopieer de **functie-Url** vanuit de portal. Het heeft de volgende notatie:

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

Toevoeg-de `name` parameter voor de query-tekenreeks. Gebruik een werkelijke naam voor de `<Enter a name here>` tijdelijke aanduiding.

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>&name=<Enter a name here>

Plak de URL in uw browser moet, en u een reactie vergelijkbaar met de volgende.

![Schermafbeelding van de Chrome-browsertabblad met test-antwoord](./media/functions-test-a-function/browser-test.png)

In dit voorbeeld is de Chrome-browser, die geschikt is voor de geretourneerde tekenreeks in XML-bestand. Andere browsers die de string-waarde weergeven.

In de portal **logboeken** venster uitvoer die vergelijkbaar is met het volgende wordt geregistreerd bij het uitvoeren van de functie:

    2016-03-23T07:34:59  Welcome, you are now connected to log-streaming service.
    2016-03-23T07:35:09.195 Function started (Id=61a8c5a9-5e44-4da0-909d-91d293f20445)
    2016-03-23T07:35:10.338 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==&name=Glenn from a browser
    2016-03-23T07:35:10.338 Request Headers = {"cache-control":"max-age=0","connection":"Keep-Alive","accept":"text/html","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T07:35:10.338 Name was provided as a query string param.
    2016-03-23T07:35:10.338 Processing User Information...
    2016-03-23T07:35:10.369 Function completed (Success, Id=61a8c5a9-5e44-4da0-909d-91d293f20445)

### <a name="test-with-postman"></a>Testen met Postman
Het aanbevolen hulpmiddel voor het testen van de meeste van uw functies wordt Postman worden geïntegreerd met de Chrome-browser. Zie voor het installeren van Postman [ophalen Postman](https://www.getpostman.com/). Postman biedt controle over veel meer kenmerken van een HTTP-aanvraag.

> [!TIP]
> Gebruik het hulpprogramma dat u meest vertrouwd met bent testen HTTP. Hier volgen enkele alternatieven voor Postman:  
>
> * [Fiddler](http://www.telerik.com/fiddler)  
> * [Paw](https://luckymarmot.com/paw)  
>
>

De functie met een aanvraagtekst testen in Postman:

1. Start Postman uit de **Apps** knop in de linkerbovenhoek van het venster Chrome-browser.
2. Kopieer uw **functie-Url**, en plak deze in Postman. Het bevat de code voor gegevenstoegang queryreeks-parameter.
3. Wijzigen van de HTTP-methode aan **POST**.
4. Klik op **hoofdtekst** > **onbewerkte**, en voeg een JSON-aanvraagtekst vergelijkbaar met het volgende:

    ```json
    {
        "name" : "Wes testing with Postman",
        "address" : "Seattle, WA 98101"
    }
    ```
5. Klik op **verzenden**.

De volgende afbeelding ziet u het voorbeeld van eenvoudige echo functie testen in deze zelfstudie.

![Schermafbeelding van de Postman-gebruikersinterface](./media/functions-test-a-function/postman-test.png)

In de portal **logboeken** venster uitvoer die vergelijkbaar is met het volgende wordt geregistreerd bij het uitvoeren van de functie:

    2016-03-23T08:04:51  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:04:57.107 Function started (Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
    2016-03-23T08:04:57.763 HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==
    2016-03-23T08:04:57.763 Request Headers = {"cache-control":"no-cache","connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T08:04:57.763 Processing user info from request body...
    2016-03-23T08:04:57.763 Processing User Information...
    2016-03-23T08:04:57.763 name = Wes testing with Postman
    2016-03-23T08:04:57.763 address = Seattle, W.A. 98101
    2016-03-23T08:04:57.795 Function completed (Success, Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)

### <a name="test-with-curl-from-the-command-line"></a>Testen met cURL vanaf de opdrachtregel
Vaak wanneer u test software, is het niet nodig om te zoeken een verder dan de opdrachtregel om te helpen bij fouten opsporen in uw toepassing. Dit is niet anders met functions testen. Houd er rekening mee dat de cURL op basis van Linux-systemen standaard beschikbaar is. Op Windows, moet u eerst downloaden en installeren van de [hulpprogramma cURL](https://curl.haxx.se/).

De functie te testen die we eerder hebt gedefinieerd, Kopieer de **functie-URL** vanuit de portal. Het heeft de volgende notatie:

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

Dit is de URL voor het activeren van uw functie. Dit testen met behulp van de cURL-opdracht op de opdrachtregel om te maken van een GET (`-G` of `--get`)-aanvraag in voor de functie:

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

Dit voorbeeld vereist een queryreeks-parameter, die kan worden doorgegeven als gegevens (`-d`) in de cURL-opdracht:

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code> -d name=<Enter a name here>

Voer de opdracht uit en ziet u de volgende uitvoer van de functie op de opdrachtregel:

![Schermafbeelding van de opdrachtprompt-uitvoer](./media/functions-test-a-function/curl-test.png)

In de portal **logboeken** venster uitvoer die vergelijkbaar is met het volgende wordt geregistreerd bij het uitvoeren van de functie:

    2016-04-05T21:55:09  Welcome, you are now connected to log-streaming service.
    2016-04-05T21:55:30.738 Function started (Id=ae6955da-29db-401a-b706-482fcd1b8f7a)
    2016-04-05T21:55:30.738 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/HttpTriggerNodeJS1?code=XXXXXXX&name=Azure Functions
    2016-04-05T21:55:30.738 Function completed (Success, Id=ae6955da-29db-401a-b706-482fcd1b8f7a)

### <a name="test-a-blob-trigger-by-using-storage-explorer"></a>Een blobtrigger testen met behulp van Storage Explorer
U kunt de functie van een blob-trigger testen met behulp van [Azure Storage Explorer](http://storageexplorer.com/).

1. In de [Azure Portal] voor uw functie-app maken een C#, F # of JavaScript triggerfunctie blob. Stel het pad voor het bewaken van de naam van de blobcontainer. Bijvoorbeeld:

        files
2. Klik op de **+** knop selecteren of maken van het opslagaccount dat u wilt gebruiken. Klik vervolgens op **Maken**.
3. Maak een tekstbestand met de volgende tekst en sla het:

        A text file for blob trigger function testing.
4. Voer [Azure Storage Explorer](http://storageexplorer.com/), en maak verbinding met de blob-container in het opslagaccount dat wordt bewaakt.
5. Klik op **uploaden** voor het uploaden van het tekstbestand.

    ![Schermafbeelding van Storage Explorer](./media/functions-test-a-function/azure-storage-explorer-test.png)

De functiecode standaard blob-trigger rapporteert de verwerking van de blob in de logboeken:

    2016-03-24T11:30:10  Welcome, you are now connected to log-streaming service.
    2016-03-24T11:30:34.472 Function started (Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)
    2016-03-24T11:30:34.472 C# Blob trigger function processed: A text file for blob trigger function testing.
    2016-03-24T11:30:34.472 Function completed (Success, Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)

## <a name="test-a-function-within-functions"></a>Een functie binnen functies testen
De Azure portal is ontworpen om te laten u testen van HTTP-functies en timer geactiveerde functies. U kunt ook functies voor het activeren van andere functies die u wilt testen maken.

### <a name="test-with-the-functions-portal-run-button"></a>Testen met de knop portal functies uitvoeren
De portal biedt een **uitvoeren** knop die u gebruiken kunt om te doen enkele testen beperkt. U kunt een aanvraagtekst opgeven met behulp van de knop, maar u kunt geen queryreeksparameters bevatten of aanvraagheaders bijwerken.

Testen van de HTTP-triggerfunctie die we eerder hebben gemaakt door het toevoegen van een JSON-tekenreeks die vergelijkbaar is met het volgende in de **aanvraagtekst** veld. Klik vervolgens op de **uitvoeren** knop.

```json
{
    "name" : "Wes testing Run button",
    "address" : "USA"
}
```

In de portal **logboeken** venster uitvoer die vergelijkbaar is met het volgende wordt geregistreerd bij het uitvoeren van de functie:

    2016-03-23T08:03:12  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:03:17.357 Function started (Id=753a01b0-45a8-4125-a030-3ad543a89409)
    2016-03-23T08:03:18.697 HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/wesmchttptriggernodejs1
    2016-03-23T08:03:18.697 Request Headers = {"connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"en-US"}
    2016-03-23T08:03:18.697 Processing user info from request body...
    2016-03-23T08:03:18.697 Processing User Information...
    2016-03-23T08:03:18.697 name = Wes testing Run button
    2016-03-23T08:03:18.697 address = USA
    2016-03-23T08:03:18.744 Function completed (Success, Id=753a01b0-45a8-4125-a030-3ad543a89409)


### <a name="test-with-a-timer-trigger"></a>Testen met een trigger voor timer
Sommige functies kunnen niet naar behoren worden getest met de hulpprogramma's die eerder is vermeld. Neem bijvoorbeeld een wachtrij trigger-functie die wordt uitgevoerd wanneer een bericht wordt verwijderd in [Azure Queue storage](../storage/queues/storage-dotnet-how-to-use-queues.md). U kunt altijd code schrijven voor een bericht in de wachtrij plaatsen en een voorbeeld hiervan in een console-project vindt u verderop in dit artikel. Er is echter een andere benadering die kunt u functies rechtstreeks te testen.  

U kunt een timertrigger geconfigureerd met een wachtrij-Uitvoerbinding. Dat de code van de timer-trigger vervolgens de testberichten naar de wachtrij schrijven kunt. In dit gedeelte leidt u door een voorbeeld.

Zie voor meer gedetailleerde informatie over het gebruik van bindingen met Azure Functions, de [referentie voor ontwikkelaars van Azure Functions](functions-reference.md).

#### <a name="create-a-queue-trigger-for-testing"></a>Maken van een wachtrijtrigger voor het testen
Om te demonstreren van deze benadering, maken we eerst een activeringsfunctie voor die we willen testen voor een wachtrij met de naam `queue-newusers`. Deze functie verwerkt de naam en adres informatie dat in Queue storage voor een nieuwe gebruiker.

> [!NOTE]
> Als u de naam van een andere wachtrij, zorg ervoor dat de naam die u gebruikt voldoet aan de [naamgeving van wachtrijen en metagegevens](https://msdn.microsoft.com/library/dd179349.aspx) regels. Anders krijgt u een foutmelding.
>
>

1. In de [Azure Portal] voor uw functie-app klikt u op **nieuwe functie** > **QueueTrigger - C#**.
2. Voer de naam van de wachtrij moet worden gevolgd door de functie wachtrij:

        queue-newusers
3. Klik op de **+** knop selecteren of maken van het opslagaccount dat u wilt gebruiken. Klik vervolgens op **Maken**.
4. Laat deze portal browservenster geopend, zodat u de logboekvermeldingen voor de standaard wachtrij functiecode sjabloon kunt bewaken.

#### <a name="create-a-timer-trigger-to-drop-a-message-in-the-queue"></a>Een timertrigger voor het verwijderen van een bericht in de wachtrij maken
1. Open de [Azure Portal] in een nieuw browservenster en navigeer naar uw functie-app.
2. Klik op **nieuwe functie** > **TimerTrigger - C#**. Voer een cron-expressie om in te stellen hoe vaak de timer code test uw wachtrij-functie. Klik vervolgens op **Maken**. Als u de test om uit te voeren van elke 30 seconden wilt, kunt u de volgende [CRON-expressie](https://wikipedia.org/wiki/Cron#CRON_expression):

        */30 * * * * *
3. Klik op de **integreren** tabblad voor uw nieuwe timertrigger.
4. Onder **uitvoer**, klikt u op **+ nieuwe uitvoer**. Klik vervolgens op **wachtrij** en **Selecteer**.
5. Houd er rekening mee de naam die u gebruikt voor de **bericht wachtrijobject**. U gebruikt deze in de functiecode timer.

        myQueue
6. Voer de naam van de wachtrij waarnaar het bericht is verzonden:

        queue-newusers
7. Klik op de **+** knop om te selecteren van het opslagaccount dat u eerder hebt gebruikt met de wachtrijtrigger. Klik vervolgens op **Opslaan**.
8. Klik op de **ontwikkelen** tabblad voor de timertrigger.
9. Als u de dezelfde wachtrij bericht objectnaam hierboven hebt gebruikt, kunt u de volgende code voor de C#-timer-functie. Klik vervolgens op **Opslaan**.

    ```cs
    using System;

    public static void Run(TimerInfo myTimer, out String myQueue, TraceWriter log)
    {
        String newUser =
        "{\"name\":\"User testing from C# timer function\",\"address\":\"XYZ\"}";

        log.Verbose($"C# Timer trigger function executed at: {DateTime.Now}");   
        log.Verbose($"{newUser}");   

        myQueue = newUser;
    }
    ```

De C#-timer-functie wordt op dit moment elke 30 seconden uitgevoerd als u het voorbeeld van de cron-expressie gebruikt. De logboeken voor de timerfunctie rapport elke uitvoering van:

    2016-03-24T10:27:02  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.004 Function started (Id=04061790-974f-4043-b851-48bd4ac424d1)
    2016-03-24T10:27:30.004 C# Timer trigger function executed at: 3/24/2016 10:27:30 AM
    2016-03-24T10:27:30.004 {"name":"User testing from C# timer function","address":"XYZ"}
    2016-03-24T10:27:30.004 Function completed (Success, Id=04061790-974f-4043-b851-48bd4ac424d1)

In het browservenster voor de wachtrij-functie ziet u elk bericht dat wordt verwerkt:

    2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
    2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"User testing from C# timer function","address":"XYZ"}
    2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)

## <a name="test-a-function-with-code"></a>Een functie met een code testen
U wilt maken van een externe toepassing of framework voor het testen van uw functies.

### <a name="test-an-http-trigger-function-with-code-nodejs"></a>Testen van een HTTP-triggerfunctie met code: Node.js
U kunt een Node.js-app gebruiken voor het uitvoeren van een HTTP-aanvraag om uw functie te testen.
Zorg ervoor dat u instelt:

* De `host` in de Aanvraagopties voor uw functie-app-host.
* De naam van uw functie in de `path`.
* Uw code voor gegevenstoegang (`<your code>`) in de `path`.

Codevoorbeeld:

```javascript
var http = require("http");

var nameQueryString = "name=Wes%20Query%20String%20Test%20From%20Node.js";

var nameBodyJSON = {
    name : "Wes testing with Node.JS code",
    address : "Dallas, T.X. 75201"
};

var bodyString = JSON.stringify(nameBodyJSON);

var options = {
  host: "functions841def78.azurewebsites.net",
  //path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9&" + nameQueryString,
  path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9",
  method: "POST",
  headers : {
      "Content-Type":"application/json",
      "Content-Length": Buffer.byteLength(bodyString)
    }    
};

callback = function(response) {
  var str = ""
  response.on("data", function (chunk) {
    str += chunk;
  });

  response.on("end", function () {
    console.log(str);
  });
}

var req = http.request(options, callback);
console.log("*** Sending name and address in body ***");
console.log(bodyString);
req.end(bodyString);
```


Uitvoer:

    C:\Users\Wesley\testing\Node.js>node testHttpTriggerExample.js
    *** Sending name and address in body ***
    {"name" : "Wes testing with Node.JS code","address" : "Dallas, T.X. 75201"}
    Hello Wes testing with Node.JS code
    The address you provided is Dallas, T.X. 75201

In de portal **logboeken** venster uitvoer die vergelijkbaar is met het volgende wordt geregistreerd bij het uitvoeren van de functie:

    2016-03-23T08:08:55  Welcome, you are now connected to log-streaming service.
    2016-03-23T08:08:59.736 Function started (Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
    2016-03-23T08:09:01.153 HTTP trigger function processed a request. RequestUri=http://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1/?code=XXXXXXXXXX==
    2016-03-23T08:09:01.153 Request Headers = {"connection":"Keep-Alive","host":"functionsExample.azurewebsites.net"}
    2016-03-23T08:09:01.153 Name not provided as query string param. Checking body...
    2016-03-23T08:09:01.153 Request Body Type = object
    2016-03-23T08:09:01.153 Request Body = [object Object]
    2016-03-23T08:09:01.153 Processing User Information...
    2016-03-23T08:09:01.215 Function completed (Success, Id=607b891c-08a1-427f-910c-af64ae4f7f9c)


### <a name="test-a-queue-trigger-function-with-code-c"></a>Een activeringsfunctie met code testen: C# #
We vermeld eerder kunt u een wachtrijtrigger testen met behulp van code verwijderen van een bericht in de wachtrij. De volgende voorbeeldcode is gebaseerd op de C#-code die zijn gepresenteerd in de [aan de slag met Azure Queue storage](../storage/queues/storage-dotnet-how-to-use-queues.md) zelfstudie. Code voor andere talen is ook beschikbaar via de koppeling.

Als u wilt testen met deze code in een console-app, moet u:

* [De opslagverbindingsreeks configureren in het bestand app.config](../storage/queues/storage-dotnet-how-to-use-queues.md).
* Geeft een `name` en `address` als parameters aan de app. Bijvoorbeeld `C:\myQueueConsoleApp\test.exe "Wes testing queues" "in a console app"`. (Deze code accepteert de naam en adres voor een nieuwe gebruiker als opdrachtregelargumenten tijdens runtime.)

Van C#-voorbeeldcode:

```cs
static void Main(string[] args)
{
    string name = null;
    string address = null;
    string queueName = "queue-newusers";
    string JSON = null;

    if (args.Length > 0)
    {
        name = args[0];
    }
    if (args.Length > 1)
    {
        address = args[1];
    }

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference(queueName);

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExists();

    // Create a message and add it to the queue.
    if (name != null)
    {
        if (address != null)
            JSON = String.Format("{{\"name\":\"{0}\",\"address\":\"{1}\"}}", name, address);
        else
            JSON = String.Format("{{\"name\":\"{0}\"}}", name);
    }

    Console.WriteLine("Adding message to " + queueName + "...");
    Console.WriteLine(JSON);

    CloudQueueMessage message = new CloudQueueMessage(JSON);
    queue.AddMessage(message);
}
```

In het browservenster voor de wachtrij-functie ziet u elk bericht dat wordt verwerkt:

    2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
    2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
    2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"Wes testing queues","address":"in a console app"}
    2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)


<!-- URLs. -->

[Azure Portal]: https://portal.azure.com
