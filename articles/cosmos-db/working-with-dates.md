---
title: Werken met datums in Azure Cosmos DB | Microsoft Docs
description: Meer informatie over het werken met datums in Azure Cosmos DB.
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: sngun
ms.openlocfilehash: d7188270ff5b1edd3b5e396be0cd5fd22e6123c4
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 07/05/2018
ms.locfileid: "37855503"
---
# <a name="working-with-dates-in-azure-cosmos-db"></a>Werken met datums in Azure Cosmos DB
Azure Cosmos DB biedt schemaflexibiliteit en geavanceerde indexeermogelijkheden via een systeemeigen [JSON](http://www.json.org) gegevensmodel. Alle Azure Cosmos DB-resources, zoals databases, containers, documenten en opgeslagen procedures worden gemodelleerd en opgeslagen als JSON-documenten. Als een vereiste voor draagbare wordt JSON (en Azure Cosmos DB) ondersteunt een kleine set basistypen: tekenreeks, getal, Booleaanse waarde, matrix, Object en Null. Echter, JSON is flexibel en kunnen ontwikkelaars en frameworks voor complexere typen met behulp van deze primitieven en samenstellen van deze objecten of-matrices. 

Naast de eenvoudige typen veel toepassingen moeten de [datum-/](https://msdn.microsoft.com/library/system.datetime(v=vs.110).aspx) type datums en een tijdstempel vertegenwoordigt. Dit artikel wordt beschreven hoe ontwikkelaars kunnen opslaan, ophalen en query uitvoeren op datums in Azure Cosmos DB met behulp van de .NET SDK.

## <a name="storing-datetimes"></a>Opslaan van datum/tijd
Standaard de [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md) serialiseert datum-/ tijdwaarden als [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874) tekenreeksen. De meeste toepassingen kunnen de tekenreeksweergave van standaard gebruiken voor datum/tijd voor de volgende redenen:

* Tekenreeksen kunnen worden vergeleken, en de relatieve positie van de datum/tijd-waarden behouden blijft wanneer ze worden getransformeerd in tekenreeksen. 
* Deze aanpak zijn vereist om een aangepaste code of kenmerken voor JSON-conversie.
* De datums die is opgeslagen in JSON zijn mens leesbaar is.
* Deze benadering kunt profiteren van Azure Cosmos DB-index voor de prestaties van snelle query's.

Bijvoorbeeld, het volgende codefragment bevat een `Order` object met eigenschappen van twee datum/tijd - `ShipDate` en `OrderDate` als een document met de .NET SDK:

    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await client.CreateDocumentAsync("/dbs/orderdb/colls/orders", 
        new Order 
        { 
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14), 
            Total = 113.39
        });

Dit document is opgeslagen in Azure Cosmos DB als volgt:

    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
    

U kunt ook kunt u de datum/tijd als Unix tijdstempels, dat wil zeggen, opslaan als een getal dat het aantal verstreken seconden sinds 1 januari 1970 vertegenwoordigt. Azure Cosmos DB van interne Timestamp (`_ts`) eigenschap deze aanpak volgt. U kunt de [UnixDateTimeConverter](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.unixdatetimeconverter.aspx) klasse voor het serialiseren van datum/tijd as-nummers. 

## <a name="indexing-datetimes-for-range-queries"></a>Datum/tijd voor de bereik-query's indexeren
Bereik-query's zijn algemene met datum-/ tijdwaarden. Als u wilt zoeken naar alle bestellingen die zijn gemaakt sinds gisteren of zoeken naar alle bestellingen in de afgelopen vijf minuten hebt verzonden, moet u bijvoorbeeld bereik-query's uitvoeren. Voor het efficiënt uitvoeren van deze query's, moet u uw verzameling voor het bereik op tekenreeksen indexeren configureren.

    DocumentCollection collection = new DocumentCollection { Id = "orders" };
    collection.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });
    await client.CreateDocumentCollectionAsync("/dbs/orderdb", collection);

U kunt meer informatie over het configureren van de indexing beleid op [Azure Cosmos DB indexeren beleid](indexing-policies.md).

## <a name="querying-datetimes-in-linq"></a>Een query uitvoeren op datum/tijd in LINQ
De SQL-SDK voor .NET ondersteunt automatisch het opvragen van gegevens die zijn opgeslagen in Azure Cosmos DB via LINQ. Bijvoorbeeld, toont het volgende fragment een LINQ-query die filters orders die zijn verzonden in de afgelopen drie dagen.

    IQueryable<Order> orders = client.CreateDocumentQuery<Order>("/dbs/orderdb/colls/orders")
        .Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
          
    // Translated to the following SQL statement and executed on Azure Cosmos DB
    SELECT * FROM root WHERE (root["ShipDate"] >= "2016-12-18T21:55:03.45569Z")

U kunt meer informatie over Azure Cosmos DB SQL-querytaal en de LINQ-provider op [uitvoeren van query's Cosmos DB](sql-api-sql-query.md).

In dit artikel hebben bekeken hoe het opslaan, index- en het opvragen van datum/tijd in Azure Cosmos DB.

## <a name="next-steps"></a>Volgende stappen
* Downloaden en uitvoeren van de [codevoorbeelden op GitHub](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples)
* Meer informatie over [SQL-query's](sql-api-sql-query.md)
* Meer informatie over [Azure Cosmos DB indexeren van beleid](indexing-policies.md)
