This tutorial teaches FIWARE users about batch commands and entity relationships.

#Contents

- [Understanding Entities and Relationships](#understanding-entities-and-relationships)
  * [Entities within a stock management system](#entities-within-a-stock-management-system)
- [Application Overview](#application-overview)
  * [Architecture](#architecture)
  * [Prerequisites](#prerequisites)
    + [Docker and Docker Compose](#docker-and-docker-compose)
    + [Cygwin for Windows](#cygwin-for-windows)
    + [Postman (Optional)](#postman-optional)
  * [Starting the containers](#starting-the-containers)
  * [Creating Several Entities at Once](#creating-several-entities-at-once)
  * [Creating a one-to-many Relationship](#creating-a-one-to-many-relationship)
  * [Reading a Foreign Key Relationship](#reading-a-foreign-key-relationship)
    + [Reading from Child Entity to Parent Entity](#reading-from-child-entity-to-parent-entity)
    + [Reading from Parent Entity to Child Entity](#reading-from-parent-entity-to-child-entity)
  * [Creating many-to-many Relationships](#creating-many-to-many-relationships)
  * [Reading from a bridge table](#reading-from-a-bridge-table)
  * [Data Integrity](#data-integrity)

# Understanding Entities and Relationships

This tutorial builds on the data created in the previous store finder example and creates and associates a series of related data entities to create a simple stock management system.

## Entities within a stock management system

Within the FIWARE platform, an entity represents the state of a physical or conceptural object which exists in the real world.

For a simple stock management system, we will only need four types of entity. The relationship between our entities is defined as shown:

![](https://fiware.github.io/tutorials.Entity-Relationships/img/entities.png)

* A **Store** is a real world bricks and mortar building. Stores would have properties such as:
  + A name of the store e.g. "Checkpoint Markt"
  + An address "Friedrichstraße 44, 10969 Kreuzberg, Berlin"
  + A phyiscal location  e.g. *52.5075 N, 13.3903 E*
* A **Shelf** is a real world device to hold objects which we wish to sell. Each shelf would have properties such as:
  + A name of the shelf e.g. "Wall Unit"
  + A phyiscal location  e.g. *52.5075 N, 13.3903 E*
  + A maximum capacity
  + An association to the store in which the shelf is present
* A **Product** is defined as something that we sell - it is conceptural object. Products would have properties such as:
  + A name of the product e.g. "Vodka"
  + A price e.g. 13.99 Euros
  + A size e.g. Small
* An **Inventory Item** is another conceptural entity, used to assocate products, stores, shelves and physical objects. It would have properties such as:
  + An assocation to the product being sold
  + An association to the store in which the product is being sold
  + An association to the shelf where the product is being displayed
  + A stock count of the quantity of the product available in the warehouse
  + A stock count of the quantity of the product available on the shelf


As you can see, each of the entities defined above contain some properties which are liable to change. A product could change its price, stock could be sold and the shelf count of stock could be reduced and so on.


# Application Overview

## Architecture

This application will only make use of one FIWARE component - the [Orion Context Broker](https://catalogue.fiware.org/enablers/publishsubscribe-context-broker-orion-context-broker). Usage of the Orion Context Broker is sufficient for an application to qualify as “Powered by FIWARE”.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep persistence of the context data it holds. Therefore, the architecture will consist of two elements:

* The Orion Context Broker server which will receive requests using NGSI
* The underlying MongoDB database associated to the Orion Context Broker server

Since all interactions between the two elements are initiated by HTTP requests, the entities can be containerized and run from exposed ports. 

![](https://fiware.github.io/tutorials.Entity-Relationships/img/architecture.png)

## Prerequisites

### Docker and Docker Compose 

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a container technology which allows to different components isolated into their respective environments. 

* To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
* To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
* To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A [YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used configure the required
services for the application. This means all container sevices can be brought up in a single commmand. Docker Compose is installed by default as part of Docker for Windows and  Docker for Mac, however Linux users will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

### Cygwin for Windows

**Cygwin** is a collection of GNU and Open Source tools which provide functionality similar to a Linux distribution on Windows. Windows users should download the tool from [here](www.cygwin.com). 


### Postman (Optional)

 **Postman** is a testing framework for REST APIs. The tool can be downloaded from [here](www.getpostman.com). 
 
The text in this tutorial uses `cUrl` commands to interact with the Orion Context Broker server, however a [Postman collection](https://raw.githubusercontent.com/Fiware/tutorials.Getting-Started/master/FIWARE%20Getting%20Started.postman_collection.json)  of commands is also available within this GitHub repository. The entire tutorial is also available directly as [Postman documentation](https://documenter.getpostman.com/view/513743/fiware-entity-relationships/RVu8g6is).



## Starting the containers

All services can be initialised from the command line by running the bash script provided within the repository:

```bash
./services start
``` 

This command will also import seed data from the previous Store Finder tutorial on startup.

>**Note:** If you want to clean up and start over again you can do so with the following command:
>
>```bash
>services stop
>``` 
>

## Creating Several Entities at Once

In the previous tutorial, we created each `STORE` entity individually, 

Lets create five shelf units at the same time. This request uses the convenience batch processing endpoint to create five shelf entities. Batch processing uses the `/v2/op/update` endpoint with a payload with two attributes - `actionType=APPEND` means we will overwrite existing entities if they exist whereas the  `entities` attribute holds an array of entities we wish to update.

To differenciate `SHELF` Entities from `STORE` Entities, each shelf has been assigned `type=SHELF`. Note that in this tutorial, the values of each entity type have been capitalized help to distinguish types from other data.

Real-world properties such as `name` and `location` have been addded as properties to each shelf.

#### Request:

```bash
curl -X POST \
  'http://localhost:1026/v2/op/update' \
  -H 'Content-Type: application/json' \
  -d '{
  "actionType":"APPEND",
  "entities":[
{
  "actionType":"APPEND",
  "entities":[
    {
      "id":"unit001", "type":"SHELF",
      "location":{
        "type":"geo:json", "value":{ "type":"Point","coordinates":[13.3986112, 52.554699]}
      },
      "name":{
        "type":"Text", "value":"Corner Unit"
      },
      "max_capacity":{
        "type":"Integer", "value":50
      }
    },
    {
      "id":"unit002", "type":"SHELF",
      "location":{
        "type":"geo:json","value":{"type":"Point","coordinates":[13.3987221, 52.5546640]}
      },
      "name":{
        "type":"Text", "value":"Wall Unit 1"
      },
      "max_capacity":{
        "type":"Integer", "value":100
      }
    },
    {
      "id":"unit003", "type":"SHELF",
      "location":{
        "type":"geo:json", "value":{"type":"Point","coordinates":[13.3987221, 52.5546640]}
      },
      "name":{
        "type":"Text", "value":"Wall Unit 2"
      },
      "max_capacity":{
        "type":"Integer", "value":100
      }
    },
    {
      "id":"unit004", "type":"SHELF",
      "location":{
        "type":"geo:json", "value":{"type":"Point","coordinates":[13.390311, 52.507522]}
      },
      "name":{
        "type":"Text", "value":"Corner Unit"
      },
      "max_capacity":{
        "type":"Integer", "value":50
      }
    },
    {
      "id":"unit005", "type":"SHELF",
      "location":{
        "type":"geo:json","value":{"type":"Point","coordinates":[13.390309, 52.50751]}
      },
      "name":{
        "type":"Text", "value":"Long Wall Unit"
      },
      "max_capacity":{
        "type":"Integer", "value":200
      }
    }
  ]
}'
```


Similarly, we can create a series of `PRODUCT` entities by using the `type=PRODUCT`.

#### Request:

```bash
curl -X POST \
  'http://localhost:1026/v2/op/update' \
  -H 'Content-Type: application/json' \
  -d '{
  "actionType":"APPEND",
  "entities":[
{
  "actionType":"APPEND",
  "entities":[
    {
      "id":"prod001", "type":"PRODUCT",
      "name":{
        "type":"Text", "value":"Beer"
      },
      "size":{
        "type":"Text", "value": "S"
      },
      "price":{
        "type":"Integer", "value": 99
      }
    },
    {
      "id":"prod002", "type":"PRODUCT",
      "name":{
        "type":"Text", "value":"Red Wine"
      },
      "size":{
        "type":"Text", "value": "M"
      },
      "price":{
        "type":"Integer", "value": 1099
      }
    },
    {
      "id":"prod003", "type":"PRODUCT",
      "name":{
        "type":"Text", "value":"White Wine"
      },
      "size":{
        "type":"Text", "value": "M"
      },
      "price":{
        "type":"Integer", "value": 1499
      }
    },
    {
      "id":"prod004", "type":"PRODUCT",
      "name":{
        "type":"Text", "value":"Vodka"
      },
      "size":{
        "type":"Text", "value": "XL"
      },
      "price":{
        "type":"Integer", "value": 5000
      }
    }
  ]
}'
```

Shelf information can be requested by making a GET request on the `entities` endpoint. For example to return the context data of the `SHELF` entity with the `id=unit001`.

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/unit001/?type=SHELF&options=keyValues'
```

#### Response:

```json
{
    "id": "unit001",
    "type": "SHELF",
    "location": {
        "type": "Point",
        "coordinates": [
            13.3986112,
            52.554699
        ]
    },
    "max_capacity": 50,
    "name": "Corner Unit"
}
```

As you can see there are currently three additional property attributes present `location`, `max_capacity` and `name`

## Creating a one-to-many Relationship

In databases, foreign keys are often used to designate a one-to-many relationship - for example every shelf is found in a single store and a single store can hold many shelving units. In order to remember this information we need to add an association relationship similar to a foreign key. Batch processing can again be used to amend the existing the `SHELF` entities to add a `store` attribute holding the relationship to each shelf.

The value of the `store` attribute corresponds to a URN associated to a `STORE` entity itself. 

The URN follows a standard format: `urn:ngsi-ld:<entity-type>:<entity-id>`

#### Request:

The following request associates three shelves to `shop1` and two shelves to `shop2`

```bash
curl -X POST \
  'http://localhost:1026/v2/op/update' \
  -H 'Content-Type: application/json' \
  -d '{
  "actionType":"APPEND",
  "entities":[
    {
      "id":"unit001", "type":"SHELF",
      "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop1"
      }
    },
    {
      "id":"unit002", "type":"SHELF",
      "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop1"
      }
    },
    {
      "id":"unit003", "type":"SHELF",
      "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop1"
      }
    },
    {
      "id":"unit004", "type":"SHELF",
      "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop2"
      }
    },
    {
      "id":"unit005", "type":"SHELF",
      "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop2"
      }
    }
  ]
}'
```

Now when the shelf information is requested again, the response has changed and includes a new property `store`, which has been added in the previous step.

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/unit001/?type=SHELF&options=keyValues'
```

#### Response:

The updated response including the `store` attribute is shown below:

```json
{
    "id": "unit001",
    "type": "SHELF",
    "location": {
        "type": "Point",
        "coordinates": [
            13.3986112,
            52.554699
        ]
    },
    "max_capacity": 50,
    "name": "Corner Unit",
    "store": "urn:ngsi-ld:STORE:shop1"
}
```


## Reading a Foreign Key Relationship

### Reading from Child Entity to Parent Entity

We can also make a request to retrieve the `store` attribute relationship information from a known `SHELF` entity by using the `options=values` setting

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/unit001/?type=SHELF&options=values&attrs=store
```

#### Response:

```json
[
    "urn:ngsi-ld:STORE:shop1"
]
```

This can be interpreted as "I am related to the `STORE` entity with the `id=shop`"

### Reading from Parent Entity to Child Entity

Reading from a parent to a child can be done using the  `options=count` setting

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/?q=store==urn:ngsi-ld:STORE:shop1&options=count&attrs=type&type=SHELF' 
```

This request is asking for the `id` of all SHELF entities associated to the URN `urn:ngsi-ld:STORE:shop1`, the response is a JSON array as shown.

#### Response:

```json
[
    {
        "id": "unit001",
        "type": "SHELF"
    },
    {
        "id": "unit002",
        "type": "SHELF"
    },
    {
        "id": "unit003",
        "type": "SHELF"
    }
]
```

In plain English, this can be interpreted as "There are three shelves in `shop1`".  The request can be altered use the `options=values` and `attrs` parameters to return specific properties of the relevant associated entities. For example the request:

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/?q=store==urn:ngsi-ld:STORE:shop1&type=SHELF&options=values&attrs=name'
```

Can be interpreted as request for *Give me the names of all shelves in `shop1`*.

#### Response:

```json
[
    [
        "Corner Unit"
    ],
    [
        "Wall Unit 1"
    ],
    [
        "Wall Unit 2"
    ]
]
```


## Creating many-to-many Relationships

Bridge Tables are often used to relate many-to-many relationships. For example, every store will sell a different range of products, and each product is sold in many different stores. 

In order to hold the context information to "place a product onto a shelf in a given store" we will need to create a new data entity `INVENTORY_ITEM` which exists to associate data from other entities. It has a foreign key relationship to 
the STORE, SHELF and PRODUCT entities and therefore requires relationship attributes called `store`, `shelf` and `product`.

Assigning a product to a shelf is simply done by creating an entity holding the relationship information and any other additional properties (such as `stock_count` and `shelf_count`)

#### Request:

```bash
curl -X POST \
  'http://localhost:1026/v2/entities/' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 0588ef62-6b5c-4d1b-8066-172d63b516fd' \
  -d '{
    "id": "0000001", "type": "INVENTORY_ITEM",
    "store": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:STORE:shop1"
    },
    "shelf": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:SHELF:unit001"
    },
    "product": { 
        "type": "Relationship",
        "value": "urn:ngsi-ld:PRODUCT:prod001"
    },
    "stock_count":{
        "type":"Integer", "value": 10000
    },
    "shelf_count":{
        "type":"Float", "value": 50
    }
}'
```



## Reading from a bridge table

When reading from a bridge table entity, the `type` of the entity must be known.

After creating at least one `INVENTORY_ITEM` we can query *Which products are sold in `shop1`?* by making the following request

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/?q=product==urn:ngsi-ld:PRODUCT:prod001&options=values&attrs=store&type=INVENTORY_ITEM' 
```

#### Response:

```json
[
    [
        "urn:ngsi-ld:STORE:shop1"
    ]
]
```


Similarly we can request *Which stores are selling `prod001`?* by altering the request as shown: 

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/?q=product==urn:ngsi-ld:PRODUCT:prod001&options=values&attrs=store&type=INVENTORY_ITEM' 
```

#### Response:

```json
[
    [
        "urn:ngsi-ld:PRODUCT:prod001"
    ]
]
```



## Data Integrity

Context data relationships should only be set up and maintained between entities that exist - in other words the URN `urn:ngsi-ld:<entity-type>:<entity-id>` should link to another existing entity within the context. Therefore we must take care when deleting an entity that no dangling references remain. Imagine `shop1` is deleted - what should happen to the associated the `SHELF` entities?

It is possible to make a request to see if any remaining entity relationship exists prior to deletion by making a request as follows

#### Request:

```bash
curl -X GET \
  'http://localhost:1026/v2/entities/?q=store==urn:ngsi-ld:STORE:shop1&options=count&attrs=type'
```


#### Request:

The response lists a series of `SHELF` and `INVENTORY_ITEM` entities - there are no `PRODUCT` entities since there is no direct relationship between product and store.

```json
[
    {
        "id": "unit001",
        "type": "SHELF"
    },
    {
        "id": "unit002",
        "type": "SHELF"
    },
    {
        "id": "unit003",
        "type": "SHELF"
    },
    {
        "id": "0000001",
        "type": "INVENTORY_ITEM"
    }
]
```

If this request returns an empty array, the entity has no associates.