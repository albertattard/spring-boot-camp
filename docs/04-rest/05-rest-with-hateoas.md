---
layout: default
title: REST with HATEOAS
parent: REST
nav_order: 5
permalink: docs/rest/with-hateoas/
---

# REST with HATEOAS
{: .no_toc }

**Hypermedia as the Engine of Application State (HATEOAS)** is a component of the REST application architecture, which allows the client interacts with a REST API entirely through the responses provided dynamically by the server.  Such an implementation of a REST Controller belong to the level 2 of the Richardson Maturity Model.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Spring support for HATEOAS

In practice the support for HATEOAS means that the REST response have to contain specific part with URL links, which dynamically show to the client the available actions. This keyword **"_links"** introduces this part of the response. It will contain one or more links. In most cases the keyword **"self"** together with the URL of the resource itself is present. Depending on the nature of the application, some other pairs of keywords and URLs will be also used.

Spring contains the module **Spring HATEOAS** which provide the support of the component from Spring. To include it into a project the dependency Spring Boot Starter HATEOAS has to be added.
```
implementation 'org.springframework.boot:spring-boot-starter-hateoas'
```

The support means that the Spring provide some wrappers to represent a resource together with URL links, and some functionality to generate such links:
* ``` EntityModel ``` - a representation model for a single resource,
* ``` CollectionModel ``` - representation model for a collection of resources
* ``` WebMvcLinkBuilder ``` - provide functionality for creating links and pointing them to controller classes


## Example of HATEOAS REST API for a single resources

One example of the HATEOAS representation for the single resource is presented below. It contains two references - to the resource itself and to the collection which the resource belong to.
``` JSON
{
   "name":"ThoughtWorks-Cologne",
   "address":"Lichtstr. 43i, 50825 Cologne, Germany",
   "phone":"+49 221 64 30 70 63",
   "email":"contact-de@thoughtworks.com",
   "_links":{
      "self":{
         "href":"http://localhost/offices/ThoughtWorks-Cologne"
      },
      "offices":{
         "href":"http://localhost/offices/"
      }
   }
}
```
&nbsp;
Here is the implementation of a method that creates the resource representation with HATEOAS.
```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @GetMapping( "/offices/{name}" )
  public EntityModel<Office> offices(final @PathVariable("name") String name) {
    Office office = service
      .findOneByName(name)
      .orElseThrow(() -> new OfficeNotFoundException("Office not found with name " + name));

    return EntityModel
      .of(office,
        linkTo(methodOn(OfficesController.class).getOffice(name)).withSelfRel(),
        linkTo(methodOn(OfficesController.class).offices()).withRel("offices"));
  }
}
```
* The method returns ``` EntityModel<Office> ``` which is a container to hold an Office object together with URL links,
* The line ``` linkTo(methodOn(OfficesController.class).getOffice(name)).withSelfRel() ``` creates the reference to the object itself,
* The line ``` linkTo(methodOn(OfficesController.class).offices()).withRel("offices")) ``` creates the reference to the collection of offices.

The test
```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should return one office with HATEOAS" )
  public void shouldReturnOneOfficeWithHateoas() {
    final String name = "ThoughtWorks Cologne";
    final Office office =
      new Office( name,
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when(service.findOneByName(eq(NAME))).thenReturn(Optional.of(office));

    MvcResult mvcResult = mockMvc.perform(get("/offices/{name}", NAME))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect( jsonPath( "$._links.self.href", containsString( "/offices/" + NAME) ))
      .andExpect( jsonPath( "$._links.offices.href", containsString( "/offices" ) ))
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).findOneByName(NAME);
    verifyNoMoreInteractions(service);
  }
}
```
* With the line ``` .andExpect( jsonPath( "$._links.self.href", containsString( "/offices/" + NAME) )) ``` the reference to the object itself is tested,
* With the line ``` .andExpect( jsonPath( "$._links.offices.href", containsString( "/offices" ) )) ``` the reference to the collections of the objects is tested.


## Example of HATEOAS REST API for a collection of resources

One example of the HATEOAS representation of the single resource is presented below. It contains two references - to the resource itself and to the collection which the resource belong to.
``` JSON
{
   "_embedded":{
      "officeList":[
         {
            "name":"ThoughtWorks-Cologne",
            "address":"Lichtstr. 43i, 50825 Cologne, Germany",
            "phone":"+49 221 64 30 70 63",
            "email":"contact-de@thoughtworks.com",
            "_links":{
               "self":{
                  "href":"http://localhost/offices/ThoughtWorks-Cologne"
               },
               "offices":{
                  "href":"http://localhost/offices/"
               }
            }
         }
      ]
   },
   "_links":{
      "self":{
         "href":"http://localhost/offices/"
      }
   }
}
```
In this example several parts for ```_links``` exist. One for the whole collection, which contains ```self``` link for the collection itself and the ```_links``` for each element of the collection.
&nbsp;
Here is the implementation of a method that creates the resource representation with HATEOAS.
```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @GetMapping("")
  public CollectionModel<EntityModel<Office>> offices() {
    List<EntityModel<Office>> offices = service
      .list()
      .stream()
      .map(office -> EntityModel.of(office,
        linkTo(methodOn(OfficesController.class).getOffice(office.getName())).withSelfRel(),
        linkTo(methodOn(OfficesController.class).offices()).withRel("offices")
      )).collect(Collectors.toList());

    return CollectionModel.of(offices,
      linkTo(methodOn(OfficesController.class).offices()).withSelfRel());
  }
}
```
* The method returns ``` CollectionModel<EntityModel<Office>> ``` which is a container to hold the collection of ``` EntityModel<Office> ``` items, this container holds also ```_links``` for the collection,
* The line ``` linkTo(methodOn(OfficesController.class).getOffice(name)).withSelfRel() ``` creates the reference to the object itself,
* The line ``` linkTo(methodOn(OfficesController.class).offices()).withRel("offices")) ``` creates the reference to the collection of offices.

The test
```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @DisplayName( "should return all offices with HATEOAS" )
  public void shouldReturnAllOfficesWithHateoas() {
    final Office office = cologneOffice();
    when( service.list() ).thenReturn( List.of( office ) );

    MvcResult mvcResult = mockMvc.perform(get("/offices"))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect( jsonPath( "_links.self.href", containsString( "/offices" ) ))
      .andExpect( jsonPath( "$..officeList[0]._links.self.href", containsInAnyOrder(containsString("/offices/" + NAME) )))
      .andExpect( jsonPath( "$..officeList[0]._links.offices.href", containsInAnyOrder(containsString( "/offices" ) )))
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();
    JsonNode officeAsJson = objectMapper.readTree(actualResponseBody).get("_embedded").get("officeList").get(0);

    assertThat(objectMapper.treeToValue( officeAsJson, Office.class )).isEqualTo(office);
    verify(service, times(1)).list();
    verifyNoMoreInteractions(service);
  }
}
```
* With the line ``` .andExpect( jsonPath( "_links.self.href", containsString( "/offices" ) )) ``` the collection's reference ```_self``` to the same collection is tested,
* With the line ``` .andExpect( jsonPath( "$..officeList[0]._links.self.href", containsInAnyOrder(containsString("/offices/" + NAME) ))) ``` the reference to the objects itself from each of items is tested,
* With the line ``` .andExpect( jsonPath( "$..officeList[0]._links.offices.href", containsInAnyOrder(containsString( "/offices" ) ))) ``` the reference to the collections of the items from each of items is tested.

## Topics for considerations

Usage of HATEOAS brings additional complexity into the application. This has to be reasonable enough from business point of view. It is a lot of big companies who don't use HATEOAS and provide public API at Level 2 of Richardson Maturity Model, for example [eBay or Uber](https://medium.com/openlight/the-trouble-with-hateoas-3ed0da733072).

There are some pros and cons of the HATEOAS, these will be taken into account, when the decision about using the HATEOAS is made.

[The advantages of HATEOAS](https://dzone.com/articles/hypermedia-driven-rest-services-with-spring-hateoa):
* Ready Made URLs - The URL structure of the API can be changed without affecting clients.
* Explorable API - The links provide the client with a list of operations that can be called based on the current state of the application.
* Workflow Style APIs - It fits well to APIs that process multiple steps as part of a user journey or workflow.

[The disadvantages of HATEOAS](https://dzone.com/articles/hypermedia-driven-rest-services-with-spring-hateoa):
* Extra Complexity for API - The API developer needs to handle the extra work of adding links to each response and providing the correct links based on the current application state, the application is more complex to build and test.
* Extra Complexity for Client - The client's application have to understand the semantic of each link and have to do extra work to parse and handle the links.
* Bigger Response Payload - The links need to be transferred from server to client, a response with hypermedia will probably need more bandwidth.
