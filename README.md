# Customer API Case Study
This is a case study to showcase API design. This includes a basic CRUD operation for customer in basic usecases . It follows [API design first](https://dzone.com/articles/an-api-first-development-approach-1) approach for development. RAML is used for design of RESTful api. The primary focus in this stage is in API design and testing the api with basic stub implementation with the help of APIkit. Further implementation can be performed in next stage. The API can be accessed [here](/src/main/api).    

## Commentary on usecase 1
**Assumptions:** It is assumed that a client may consumes the customer API each 5 minutes in order to have a copy of customers in its own database.  Depending on other constraints any of the following design considerations can be taken in to account.

1. HTTP caching:
This is mostly effective if the number of customers is low or the number of customer updates is low  in certain times of the day such as midnight.   
To enable it, conditional get and revalidation is used. The server sends the following http header to the client:

```
Cache-Control: private, no-store, max-age=300 
ETag: "asnasdsdd"

```
The client sends the following header with the same Etag in next requests:

```
If-None-Match
ETag:  "asnasdsdd"
```
The server checks the Etag and responds with 304 status code if not modified and  200 with new represntation if modified.  new Etag is also returned to the client for subsequent requests if modified. 
More information on this can be found [here](https://dennis-xlc.gitbooks.io/restful-java-with-jax-rs-2-0-2rd-edition/en/part1/chapter11/caching.html)

**NOTE:** This caching is considered in api design with Cacheable Trait.

2. Server side caching:
the server may implement a cache management to avoid database hits and achieve network efficiency if the number of customer records is high or the api is accessed more frequently.

3. More efficient incremental client/server synchronization design:   
If the number of customer records is high, it is not efficient to send all the customer data through the network and the client most likely requires more than 5 minutes to update its own database.    
In this scenario it would be better to consider design of incremental synchronization between the client and server. To do this, the server has to persist and cache any modification happens to customer records and sends a last modified date with the description of change and the changed records to the client. The client applies the changes to its own database. The client sends the saved last modified date to the server next time accesses to the server.  
An extension to this scenario could be considering other ways of synchronization such as file based integration with batch processing or database synchronization if the same database is used by both client and server or both databases provide ETL mechanism.

**Note:** Different aspects of design and implementation of the usecase is discussed here. It does not mean that these solutions should be used out of the box. Each of the solutions has its own implementation costs and should be used on a conditional basis.

### Commentary on usecase 2
**Assumptions:** The key considerations is mobile and efficient network use  
**Considerations:**
1. To design this a new resource called customerId is defined to access the resource directly with more ease. 
2. Partial trait is defined so that the client be able to requests the necessary fields in response. This yields to network effeciency although customer object is small itself. 
3. Cacheable trait as discussed in usecase 1 is defined to avoid unnecessary data transfer over the network when a customer is not modified in GET request.
4. Pageable trait is defined to support paging for effecient customers call over the network.

### Commentary on usecase 3
**Assumptions:** Simple extension of the API to support future resources such as orders and products
**Considerations:**
1. Products and orders will most likely have the same pattern as customers such as following:
```
/customers:
  get:
  post:
  /{customerId}:
    get:
    put:
    delete:

```
To support this pattern, the  **"collection/collection-item"** pattern is properly implemented.    
2. Common Schemas and Traits such as Cacheable, Secured and Pageable are also refactored to separate files to achieve reusability. This way each trait can applied as required.  
3. Separate packaging is defined for each resource such as resourceTypes. Sub-packaging may be considered for further extension.      
By doing the above steps, adding products and orders could be as simple as the following : 
```
/products:
  get:
  post:
  	is [secured]
  /{productId}:
    get:
	is [cacheable, partial]
    put:
    	is [secured]
    delete:
    	is [secured]
```
**Note:** If any requirements differs from the base design, it can be implemented by overwriting any of the map's elements as RAML supports this kind of inheritance. 

## Design decistions and considerations
#### Security
* Simple authentications by providing an access token for write methods is implemented. To enhance security, **Ouath2** standard is recommended. 
* Transport protocol is limited to Https to ensure message confidentiality and integrity
* Other security concerns such as access control, rate of usage can be considered later if required.

#### Error handling and response codes
* Number of HTTP response codes is purposefully limited to 8 familiar HTTP codes for ease of use by developers. To achieve more simplicity, the http codes can even be reduced to the minimum 200, 400 and 500.
* A result.schema is designed to provide meaningful and additional information about error codes especially for 400 HTTP status code. A field called moreInfo is considered to redirect the developers to the documentation portal

#### Naming conventions
* Nouns are used instead of verbs
* Javascript camelCase convention for naming attributes is used which is more developer friendly 
* Whenever appropriate, common naming  is favored. For example, pageSize and perPage renamed to limit and offset because it is more common in leading databases such as MySql 

### Filtering, Sort and Paging
* Filtering and sort is supported by designing Searchable and Sortable traits so that the client be able to do filtering and sort if required.
* Paging is supported by Pageable trait to reduce the number of records over the network. Especially if the mobile client requires to access customers, products and orders. 

### Other considerations
* Whenever meaningful, validation by providing limits and proper default is considered.
* Versioning is used to support next releases.
* Message formats are limited to JSON although other formats can be supported. 
* For exceptional scenarios where the clients do not support http codes other than 200 such as some Adobe flash clients, an optional parameter called suppress_response_codes is considered in Suppressable trait. When suppress_response_codes is set to true, the HTTP response is always 200 and detailed information is supported by verbose message such as the one described in Error handling section.

### Implementation Considerations
Basic stub implementation is provided with help of APIkit and Anypoint Studio. For further implementation the following can be considered.
* Caching can be implemented by builtin caching provided in Mule enterprise edition or other caching such as EHcache can be implemented with open source Mule
* Proper exception handling should be used in Mule to support the detailed error codes described in Error handling section. 
* JAX-RS can be used for RESTful api implementation
* Proper Validation and logging should be considered.
* Mule Spring security can be used for security.  
* Programming to interfaces such as CustomerService with help of Spring and Mule Spring support should be considered.
* DAO pattern can be used for data access. Hibernate is a good candidate for implementing the pattern.

## Testing the API designs
The project can be imported to Anypoint Design Center and tested by mocking service provided by the design center. For more information visit [Anypoint Design Center](https://www.mulesoft.com/platform/anypoint-design-center). 
Alternatively, the project can be built and run in Anypoint Studio. To do that, follow the instruction provided by this [APIkit documentation](https://docs.mulesoft.com/apikit/).

## Authors

* **Reza Arifi** 

	
