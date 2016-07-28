# E-warehouse API
HC Distributie biedt een krachtige API voor het E-warehouse systeem. De E-warehouse API is ontwikkeld zodat derde partijen de vrijheid krijgen om een deel van de E-warehouse functionaliteiten te gebruiken, automatiseren en verder uit te breiden. Doormiddel van een REST API is het mogelijk data uit te lezen en aan te passen.

Op dit moment is de E-warehouse API in `v1.0`. 

## Authenticatie
Bij het ontwikkelen van de E-warehouse API is gekozen voor een authenticatie-vorm waarbij gebruik wordt gemaakt met 24-uur geldige validation-keys, die bij iedere request in de Header van de request meegegeven dienen te worden. De gegenereerde validation-key dient meegegeven in de header, als `validation_key`.

### `POST` Validation-Key opvragen
Bij het aanvragen van een validation key zijn 3 verschillende velden nodig: `UserId`, `Timestamp` en `Hash`. 

Hierbij bestaat het veld `UserId` uit het Id van uw gebruiker. Deze kan u vinden door binnen E-warehouse rechts-bovenin op uw naam te klikken en vervolgens naar *Account* te navigeren. Hier zult u uw `UserId` vinden.
```json
VOORBEELD: 
POST /api/Authentication
{
    "UserId": 800,
    "Timestamp": "#7/4/2016 12:00:00 AM#",
    "Hash": "KhaPn/NENjCgqy68PVpgnA=="
}
```
`Timestamp` bestaat uit een DateTime waarde van de huidige tijd. Deze dient vervolgens ook gebruikt en meegegeven te worden in de `Hash`. Een voorbeeld van een 'op deze manier geformateerde DateTime' is: `#07/23/2016 12:00:00 AM#`.

De variabele `Hash` bestaat uit een combinatie van de meegeven `Timestamp`, uw Customer-ID (*het **10000** nummer na de 100- dat u ingeeft bij het inloggen op E-warehouse*), uw gebruikersnaam en uw wachtwoord. Er wordt hierbij **géén** gebruik gemaakt van koppeltekens of spaties als seperatie tussen de variabelen, ze zijn dus opvolgend. Deze variabelenreeks dient ge-MD5 hashed te worden en daarna geconverteerd naar een `Base64 String`- dit resultaat wordt dan meegestuurd als variabelen `Hash`. Een voorbeeld van een niet-gehashde string is: `#7/4/2016 12:00:00 AM#10001api.creatorveiligheid2016` naar `KhaPn/NENjCgqy68PVpgnA==`.
```json
VOORBEELD RESPONSE:
{
    "Status": "success",
    "StatusCode": 200,
    "Description": "Successfully requested an authenticationkey.",
    "Content": {
        "UserId": 800,
        "Timestamp": "2016-07-04T00:00:00",
        "Hash": "KhaPn/NENjCgqy68PVpgnA==",
        "ValidationKey": "ef481695-4f8f-4b0e-ad19-70fb498c4e44"
    }
}
```

| ARGUMENTS          | |
| -------------:    |:-------------|
| `UserId`          | The Id corresponding to your user-account |
| `Timestamp`       | A DateTime of the request, this will be used in the `Hash` to verify it     |
| `Hash`            | Combination of `Timestamp`, Customer-ID, Username and Password      |


## Orders
Orders kunnen opgevraagd worden en ook kunnen bepaalde velden van een order gewijzigd worden.

### `GET` Order opvragen
Een enkele order kan aangevraagd worden door een `GET`-request te sturen naar `/api/Orders/<OrderID>`. Let hierbij wel op dat de aangevraagde order moet toebehoren aan u of aan een van uw shops.
```JSON
VOORBEELD:
GET /api/Orders/200100000
```
```JSON
VOORBEELD RESPONSE:
{
    "Status": "success",
    "StatusCode": 200,
    "Description": "Successfully requested order #200100000.",
    "Content": {
        "OrderID": 200100000,
        "ShopID":10001,
        "CustomerID": 10001,
        "Reference": "WS 10000000",
        "DateTime": "2016-07-02T12:00:00",
        "OrderReceiver": {
            "Name": "T Visser",
            "CompanyName": "HCGroup",
            "Address": "Graafschaphornelaan 137A",
            "PostalCode": "6001  AC",
            "City": "Weert",
            "Country": "NL",
            "PhoneNumber": "+31 (0)495 788 118",
            "EmailAddress": "t.visser@hcgroup.nl"
        },
        "ExpectedDelivery": "2016-07-04T00:00:00",
        "DefiniteDelivery": "2017-07-04T00:00:00",
        "OrderDetails": [
            {
                "OrderRowIndex": 1,
                "ProductID": 5000200,
                "Quantity": 20,
                "QuantityDelivered": 20,
                "ExpectedDelivery": "2017-07-04T00:00:00",
                "IsReversed": false,
                "Remark": ""
            },
            {
                "OrderRowIndex": 0,
                "ProductID": 5000100,
                "Quantity": 5,
                "QuantityDelivered": 5,
                "ExpectedDelivery": "2017-07-04T00:00:00",
                "IsReversed": false,
                "Remark": ""
            }
        ],
        "Weight": 2000,
        "NumberOfPackages": 1,
        "Packaging": [
            {
                "PackagingName": "E3 - karton - enkelgolf",
                "MaxWeight": 5000,
                "Width": 400,
                "Height": 250,
                "Depth": 300,
                "ArticleID": 10156
            }
        ],
        "ShippingMethod": "Postnl",
        "Status": {
            "OrderStatusId": 5,
            "Description": "Afgerond",
            "Progress": 100
        },
        "Remark": "Order automatisch aangelegd door SEOShop.",
        "CustomerReference": "ORD20200",
        "TrackAndTraceURL": "",
        "NumberOfPallets": 0,
        "SendDate": "2016-07-02T00:00:00",
        "IsInvoiced": false
    }
}
```

### `GET` Orders opvragen
Het is ook mogelijk om in bulk de meest recente orders op te vragen. Deze kunnen doormiddel van verschillende argumenten gefilterd worden.
```JSON
VOORBEELD:
GET /api/Orders
```

Het is ook mogelijk om bepaalde argumenten mee te geven tijdens het opvragen van bulk-orders:
```JSON
VOORBEELD:
GET /api/Orders?count=20&offset=60&status=4
```
| ARGUMENTS          | |
| -------------:    |:-------------|
| `count`          | The amount of orders to retrieve. (Max: 500) |
| `offset`       | Used for paging orders, skips to the specified index       |
| `status`       | A filter on the order-status       |

### `POST` Orders aanpassen
Bij het updaten van orders is het maar mogelijk om bepaalde velden te updaten wanneer de order een bepaalde status nog niet bereikt heeft. Zo is het natuurlijk niet mogelijk om de geadresseerde aan te passen wanneer de order al verzonden is. 

Bij het updaten wordt een `array` van Order-objecten verwacht. Bij ieder Order-object is het verplicht een `OrderID` mee te geven, aangezien deze refereerd naar het aan-te-passen order. Het aanpassen van `OrderReceiver` data kan alleen wanneer de `Status.Progress` zich onder de 40% bevindt. Het aanpassen van `OrderDetails` data kan alleen wanneer de `Status.Progress` zich onder de 10% bevindt.
```json
VOORBEELD:
POST /api/Orders
[
    {
        "OrderID": 200100000,
        "OrderReceiver": {
            "Name": "T Visser",
            "CompanyName": "HCGroup",
            "Address": "Graafschaphornelaan 137A",
            "PostalCode": "6001  AC",
            "City": "Weert",
            "Country": "NL",
            "PhoneNumber": "+31 (0)495 788 118",
            "EmailAddress": "t.visser@hcgroup.nl"
        },
        "OrderDetails": [
            {
                "ProductID": 5000100,
                "Quantity": 5,
                "IsReversed": false,
                "Remark": ""
            },
            {
                "ProductID": 5000200,
                "Quantity": 20,
                "IsReversed": false,
                "Remark": ""
            },
            {
                "ProductID": 5000300,
                "Quantity": 20,
                "IsReversed": false,
                "Remark": ""
            },
        ],
        "Remark": "Order aangepast doormiddel van de API."
    },
    {
        "OrderID": 200100001,
        "OrderReceiver": {
            "Name": "T Visser",
            "CompanyName": "HCGroup",
            "Address": "Maaspoort 268",
            "PostalCode": "6001 BN",
            "City": "Weert",
            "Country": "NL",
            "PhoneNumber": "+31 (0)495 788 118",
            "EmailAddress": "t.visser@hcgroup.nl"
        },
        "Remark": "Ontvanger is verhuisd."
    }
]
```

Een aantal punten om rekening mee te houden bij het aanpassen van een order:

- Om een OrderDetail aan te passen dient een bestaand ProductID gebruikt te worden als referentie naar de OrderDetail.

- Een nieuwe OrderDetail kan aangelegd worden door een ander ProductID te gebruiken.

- Het is niet mogelijk om OrderDetails definitief te verwijderen, het is echter wel mogelijk de OrderDetail te storneren.

Het eerste Order-Object in het voorbeeld bevat alle aanpasbare velden.

### `POST` Orders aanmaken
Orders kunnen aangemaakt worden door een Order-object zónder ID te voorzien. Deze kunnen in bulk geleverd worden. Hierbij zijn de volgende velden vereist: `ShopID`, `OrderReceiver`, `OrderDetails` (_Hierbij is tenminste 1 OrderDetail verplicht_). Orders kunnen (_op dit moment_) niet verwijderd worden. Wel kunnen alle artikelen binnen een Order gestorneerd worden. In het voorbeeld zijn alle velden terug te zien.

```json
VOORBEELD:
POST /api/Orders
[
    {
        "Reference": "ORDER REF #1000000",
        "ShopID": 10001,
        "OrderReceiver": {
            "Name": "T Visser",
            "CompanyName": "HCGroup",
            "Address": "Graafschaphornelaan 137A",
            "PostalCode": "6001  AC",
            "City": "Weert",
            "Country": "NL",
            "PhoneNumber": "+31 (0)495 788 118",
            "EmailAddress": "t.visser@hcgroup.nl"
        },
        "OrderDetails": [
            {
                "ProductID": 5000100,
                "Quantity": 10,
                "isReversed": false,
                "Remark":" De velden ProductID en Quantity zijn verplicht, isReversed en Remark zijn optioneel."
            },
            {
                "ProductID": 5000200,
                "Quantity": 20
            }
        ],
        "Remark": "Order automatisch aangelegd door de API.",
        "CustomerReference": "ORD20200"
    }
]
```

## Stock
De voorraad van producten worden geheel verwerkt en gemodereerd door het E-warehouse systeem. De voorraad kan echter wel uitgelezen worden.

### `GET` Stock opvragen
Het opvragen van een enkele product stock gaat doormiddel van een `GET`-request.
```JSON
VOORBEELD:
GET /api/Stock/5000100
```
Bij slave producten wordt de voorraad van de master teruggekoppeld. Bij de combi-artikelen worden de mogelijke hoeveelheid combinaties als voorraad teruggekoppeld.
```JSON
VOORBEELD RESPONSE:
{
    "Status": "success",
    "StatusCode": 200,
    "Description": "Successfully requested stock-data on product #5000100.",
    "Content": {
        "ProductID": 5000100,
        "StockAmount": 120
    }
}
```

## Products
Ook is het mogelijk om bepaalde data van producten aan te passen en op te vragen. Producten zijn alle fysiek-aanwezige producten in het warehouse. 

### `GET` Product opvragen
Een enkele product kan aangevraagd worden door een `GET`-request te sturen naar `/api/Products/<ProductID>`. Let hierbij wel op dat het aangevraagde product moet toebehoren aan u of aan een van uw shops.
```JSON
VOORBEELD:
GET /api/Products/5000100
```

```JSON
VOORBEELD RESPONSE:
{
    "Status": "success",
    "StatusCode": 200,
    "Description": "Successfully requested product #5000100.",
    "Content": {
        "ProductID": 5000100,
        "ShopID": 10001,
        "Name": "Test Artikel",
        "Description": "Een test-artikel aangemaakt voor systeemtesting.",
        "VATType": "Low",
        "StockAmount": 100,
        "StockReserved": 0,
        "CustomID": "CSTM 00001",
        "Measurement": {
            "Weight": 200,
            "Width": 10,
            "Height": 20,
            "Depth": 5
        },
        "Barcode": "",
        "LastUpdate": "2016-07-01T00:00:00",
        "EAN": "E-0000000001",
        "SKU": "S-0000000001",
        "ArticleCode": "ART-50000",
        "Packaging": {
            "PackagingName": "Onbekend"
        },
        "isSlaveProduct": false,
        "hasMinimumStock": true,
        "MinimumStock": 10,
        "isCombinationProduct": false
    }
}
```

### `GET` Products opvragen
Een grote hoeveelheid producten aanvragen is mogelijk door een `GET`-request te sturen naar /api/Products. Hierbij worden de meest recent-toegevoegde producten teruggegeven.
```JSON
VOORBEELD:
GET /api/Products
```

Het is ook mogelijk om bepaalde argumenten mee te geven tijdens het opvragen van bulk-Producten:
```JSON
VOORBEELD:
GET /api/Orders?count=100&offset=500&sku=SKU-7081
```
| ARGUMENTS          | |
| -------------:    |:-------------|
| `count`          | The amount of products to retrieve. (Max: 500) |
| `offset`       | Used for paging products, skips to the specified index       |
| `sku`       | Search for products with specified SKU       |
| `ean`       | Search for products with specified EAN       |
| `articlecode`       | Search for products with specified ArticleCode       |

### `POST` Products aanpassen
Het aanpassen van producten biedt relatief meer mogelijkheden dan bijvoorbeeld het aanpassen van een order. In de voorbeeld `POST` worden alle verschillende aan-te-passen velden gebruikt in het *eerste voorbeeld*, behalve de implementatie van CombinationOfProducts, aangezien een slave-product niet zowel een combination-product kan zijn. Let wel op dat het van belang is een ProductID te gebruiken, aangezien deze dient te refereren naar het product.
```JSON
VOORBEELD:
POST /api/Products
[
    {
        "ProductID": 5000100,
        "Name": "Groter Test Product",
        "Description": "Een groot test product.",
        "VATType": "High",
        "CustomID": "CSTM 20000",
        "Measurement": {
            "Weight": 500,
            "Width": 20,
            "Height": 40,
            "Depth": 10
        },
        "Barcode": "8181818181",
        "EAN": "E-0000000005",
        "SKU": "S-0000000005",
        "ArticleCode": "ART-50000",
        "isSlaveProduct": true,
        "MasterProductID": 5000150,
        "hasMinimumStock": true,
        "MinimumStock": 20,
        "isCombinationProduct": false
    }, 
    {
        "ProductID": 5000200,
        "Name": "Combinatie-product",
        "EAN": "E-0000000010",
        "ArticleCode": "ART-60000",
        "isCombinationProduct": true,
        "CombinationOfProducts": [
            {
                "ProductID": 5000220,
                "AmountOfProduct": 5
            },
            {
                "ProductID": 5000240,
                "AmountOfProduct": 10
            }
        ]
    }, 
    {
        "ProductID": 5000300,
        "Name": "Nog Groter Test Product",
        "Description": "Groter dan dit product gaat het niet!",
        "Measurement": {
            "Weight": 1000,
            "Width": 500,
            "Height": 400,
            "Depth": 100
        },
        "SKU": "S-0000000015",
        "ArticleCode": "ART-65000"
    }
]
```

| ARGUMENTS          | |
| -------------:    |:-------------|
| `Name`          | The name of the product |
| `Description`       | A description on the product|
| `VATType`       | Type of VAT accounted over te product (`Low`, `High` and `Packaging` are possible values)|
| `CustomID` | A custom reference-id able to be set by the customer|
| `Measurements`.`*` | All measurement-values of the product (dimensions and weight)|
| `Barcode` | The barcode of a product. **WARNING**: This value is used for orderpicking a product, when this value is set invalid it may cause you to be charged / some orders might get lost. Keep this in mind when modifying the barcode|
| `EAN` | The EAN code of the product|
| `SKU` | The SKU code of the product|
| `ArticleCode` | The article-code of the product|
| `isSlaveProduct` | Is the product a slave product. If `isSlaveProduct` is `true`, the parameter `MasterProductID` is required to successfully update the product|
| `MasterProductID` | The Id of the Master-product. `isSlaveProduct` needs to be set to `true` to update this field|
| `hasMinimumStock` | Select if the product needs to send a warning when the product reaches a specific amount. If `hasMinimumStock` is `true`, the parameter `MinimumStock` is required to successfully update the product|
| `isCombinationProduct` | Select if this product is seen as a combination-product. If `isCombinationProduct` is `true`, the object-parameter `CombinationOfProducts` needs to include atleast a single valid object|
| `CombinationOfProducts` | An object-array of all products in the combination, with their corresponding amounts (_see the second example for valid implementation_)|


### `POST` Products aanmaken
Producten kunnen aangemaakt worden door een Product-object zónder ID te voorzien. Deze kunnen in bulk geleverd worden. Let er wel op dat wanneer er een syntax/error bij een van de objecten is, hij er géén zal toevoegen (alleen bij een compleet succesvolle transactie worden alle objecten daadwerkelijk aangemaakt). In het voorbeeld ziet u de 3 verschillende producten terug (*Simpel-, Combinatie- en Slaveproducten*).
```JSON
VOORBEELD:
POST /api/Products
[
   {
      "ShopID": 10001,
      "Name": "Nieuw Simpel Product",
      "Description": "Recent toegevoegd product. Volledig nieuw in de collectie.",
      "VATType": "High",
      "CustomID": "CSTM 10500",
      "Measurement": {
         "Weight": 200,
         "Width":120,
         "Height":120,
         "Depth":60
      },
      "Barcode": "",
      "EAN": "E-0012300001",
      "SKU": "S-0041900001",
      "ArticleCode": "ART-70000",
      "isSlaveProduct": false,
      "hasMinimumStock": true,
      "MinimumStock": 10,
      "isCombinationProduct":false
   },
   {
      "ShopID": 10001,
      "Name": "Nieuw Slave Product",
      "Description": "Een groot test product.",
      "VATType": "High",
      "CustomID": "CSTM 20000",
      "Measurement": {
         "Weight":0,
         "Width":0,
         "Height":0,
         "Depth":0
      },
      "Barcode": "",
      "EAN": "E-0000000005",
      "SKU": "S-0000000005",
      "ArticleCode": "ART-50000"
      "isSlaveProduct": true,
      "MasterProductID": 5000100,
      "hasMinimumStock": false,
      "isCombinationProduct":false
   },
   {
      "ShopID": 10001,
      "Name": "Nieuw Combi Product",
      "Description": "Recent toegevoegd Combi Product.",
      "VATType": "High",
      "CustomID": "CSTM 10520",
      "Measurement": {
         "Weight":0,
         "Width":0,
         "Height":0,
         "Depth":0
      },
      "Barcode": "",
      "EAN": "E-0033300001",
      "SKU": "S-0046900001",
      "ArticleCode": "ART-80000",
      "isSlaveProduct": false,
      "hasMinimumStock": false,
      "isCombinationProduct":true,
      "CombinationOfProducts":[
         {
            "ProductID": 5000100,
            "AmountOfProduct": 1
         },
         {
            "ProductID": 5000200,
            "AmountOfProduct": 5
         }
      ]
   }
]
```
Het aanpassen van artikelen en het toevoegen van nieuwe kan samen in bulk gedaan worden. 

## Retouren
Het is mogelijk om alle retour-aanvragen in te zien, met de bijbehorende data. Deze zijn op dit moment niet aanpasbaar, maar zijn wel volledig uitleesbaar. De retourregels worden aan een retour toegevoegd wanneer de retour-producten fysiek zijn ontvangen.

### `GET` Retour opvragen
Een enkele retour kan aangevraagd worden doormiddel van een `GET` request, naar de URL met het Retour-Id.
```JSON
VOORBEELD:
GET /api/Returns/50000100
```

```JSON
VOORBEELD RESPONSE:
{
  "Status": "success",
  "StatusCode": 200,
  "Description": "Successfully requested order-return #50000100",
  "Content": {
    "ReturnID": 50000100,
    "OrderID": 200000200,
    "ReturnDate": "2016-07-24T00:00:00",
    "isPickReturn": false,
    "Reason": "Niet naar verwachting",
    "Remark": "geweigerd",
    "Location": {
      "Country": "NL"
    },
    "Status": "Verwerkt en Retour Voorraad",
    "ReturnDetails": [
      {
        "ProductID": 5000100,
        "Amount": 10,
        "Status": "Terug op Voorraad"
      }
    ],
    "WarehousingDate": "2016-07-25T00:00:00",
    "TrackAndTrace": ""
  }
}
```

### `GET` Retours opvragen
Ook een Bulk aan retours kan worden opgevraagd door een `GET` request. Hierbij kan worden gepagineerd door de variabelen `offset` te gebruiken. Ook kan de hoeveelheid resultaten worden gewijzigd doormiddel van de parameter `count` (_zie het voorbeeld voor de implementatie van beide_)
```JSON
VOORBEELD:
GET /api/Returns?count=3&offset=15
```

```JSON
VOORBEELD RESPONSE:
{
  "Status": "success",
  "StatusCode": 200,
  "Description": "Successfully requested returns. Total of 3 return(s) found.",
  "Content": [
    {
      "ReturnID": 50000105,
      "OrderID": 200000210,
      "ReturnDate": "2016-07-24T00:00:00",
      "isPickReturn": false,
      "Reason": "Niet naar verwachting",
      "Remark": "",
      "Location": {
        "Country": "NL"
      },
      "Status": "Verwerkt en Retour Voorraad",
      "ReturnDetails": [
        {
          "ProductID": 5000100,
          "Amount": 10,
          "Status": "Terug op Voorraad"
        },
        {
          "ProductID": 5000200,
          "Amount": 5,
          "Status": "Terug op Voorraad"
        }
      ],
      "WarehousingDate": "2016-07-25T00:00:00",
      "TrackAndTrace": ""
    },
    {
      "ReturnID": 50000104,
      "OrderID": 200000205,
      "ReturnDate": "2016-07-24T00:00:00",
      "isPickReturn": false,
      "Reason": "Niet naar verwachting",
      "Remark": "Amazon Retour",
      "Location": {
        "Country": "NL"
      },
      "Status": "Verwerkt en Retour Voorraad",
      "ReturnDetails": [
        {
          "ProductID": 5000100,
          "Amount": 10,
          "Status": "Terug op Voorraad"
        }
      ],
      "WarehousingDate": "2016-07-25T00:00:00",
      "TrackAndTrace": ""
    },
    {
      "ReturnID": 50000103,
      "OrderID": 200000200,
      "ReturnDate": "2016-07-23T00:00:00",
      "isPickReturn": false,
      "Reason": "Niet naar verwachting",
      "Remark": "Via E-warehouse aangemaakt zonder label",
      "Location": {
        "Country": "NL"
      },
      "Status": "Verwerkt en Retour Voorraad",
      "WarehousingDate": "1900-01-01T00:00:00",
      "TrackAndTrace": ""
    }
  ]
}
```
