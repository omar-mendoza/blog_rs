# Paging and Sorting with Spring Data JPA

## Motivation

When we develop applications many times we have to execute searches/queries to obtain specific data from our database(s). 
In our database(s), maybe, we have a lot of information we want to show or share with one or more of our applications or sometimes with external consumers 
through APIs. So, an important consideration to do that is how we can show/share those data efficiently. A way to improve how we share data is paging 
those data, in this way the applications (web app, mobile app, etc.) can consume those data in small parts and not all the information is 
loaded at the same time.

## Spring Data and Spring Data JPA

In this post I'll show how can we make paging and sorting data with one of the hottest technologies today 
[Spring Data JPA](https://spring.io/projects/spring-data-jpa). **Spring Data JPA** is one of the 
[Spring Data](https://spring.io/projects/spring-data) data access layer projects which make easy create JPA based reposotories.

The purpose of **Spring Data** according to his page is:

*"provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.\
It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies."*

### Spring Data JPA Features
* Sophisticated support to build repositories based on Spring and JPA
* Support for Querydsl predicates and thus type-safe JPA queries
* Transparent auditing of domain class
* Pagination support, dynamic query execution, ability to integrate custom data access code
* Validation of @Query annotated queries at bootstrap time
* Support for XML based entity mapping
* JavaConfig based repository configuration by introducing @EnableJpaRepositories.

## Setup
First, we need to create our domain model for our data. It consist of an Hero **Entity**. **Hero** class have two properties,
realName and heroName

``` java
@Entity
public class Hero {

	@Id
   @GeneratedValue(strategy=GenerationType.AUTO)
	private Integer id;
	
	private String realName;

	private String heroName;
	
	public Hero() {};
	
	public Hero(String realName, String heroName) {
		this.realName = realName;
		this.heroName = heroName;
	}
	
	public String getRealName() {
		return realName;
	}
	public void setRealName(String realName) {
		this.realName = realName;
	}
	public String getHeroName() {
		return heroName;
	}
	public void setHeroName(String heroName) {
		this.heroName = heroName;
	}
	
	
}
```

## Spring Data Repository

To be able to show/share our data, we need to retrieve the data first, so, we need to create a simple interface who extends
**PagingAndSortingRepository** (*HeroRepository*)

``` java
public interface HeroRepository extends PagingAndSortingRepository<Hero, Long> {
}
```

Having extended from [PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html) we have the following methods (without coding anything)

``` java
Iterable<Hero> findAll(Pageable pageable);
Iterable<Hero> findAll(Sort sort);
```
## Paging and Sorting
After having created our repository, which extends *PagingAndSortingRepository*, we can recover our data as follow

``` java
Pageable pageable = PageRequest.of(0, 20);
Page<Hero> heroes = heroRepository.findAll(pageable);
```

First argument of **PageRequest** is the page number and the second is the size of the set. So, how we can see
*heroRepository.findAll(pageable);* returns a *Page\<Hero>* instance. *Page\<Hero>* instance contains among other information the 
list of heroes and the total of avalaible pages

``` json
{
   "content":[
      {
         "realName":"Barbara Gordon",
         "heroName":"Batgirl"
      },
      {
         "realName":"Barry Allen",
         "heroName":"Flash"
      },
      {
         "realName":"Bruce Wayne",
         "heroName":"Batman"
      },
      {
         "realName":"Hal Jordan",
         "heroName":"Green Lantern"
      },
      {
         "realName":"Oliver Queen",
         "heroName":"Green Arrow"
      }
   ],
   "pageable":{
      "sort":{
         "sorted":true,
         "unsorted":false,
         "empty":false
      },
      "offset":0,
      "pageNumber":0,
      "pageSize":20,
      "paged":true,
      "unpaged":false
   },
   "last":true,
   "totalElements":5,
   "totalPages":1,
   "size":20,
   "number":0,
   "sort":{
      "sorted":true,
      "unsorted":false,
      "empty":false
   },
   "numberOfElements":5,
   "first":true,
   "empty":false
}
```

In the same way, we can sort the data, as follow
``` java
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName"));
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName").ascending());
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName").descending());
```
We can combine paging and sorting too
``` java
Pageable pageable = PageRequest.of(0, 10, Sort.by("heroName"));
Page<Hero> heroes = heroRepository.findAll(pageable);
```

## Custom Paging and Sorting

**Spring Data JPA** provide us a set of methods for basics searchs operations but sometimes it is not enough. So, we can create some 
extra methods to recover our neccesaries data. To do this we can create methods into our repository interface. For example, if we want the 
list of all the heroes with the same real or hero name, our methods looks like 

``` java
public Iterable<Hero> findAllByRealName(String heroName);
public Iterable<Hero> findAllByHeroName(String heroName);
```

Doing that **Spring Data JPA** knows what we want. We want all the heroes with an specific real-name or hero-name. So, the name of 
the method defines our query

``` java
Pageable pageableRealName = PageRequest.of(0, 10, Sort.by("heroName"));
Pageable pageableHeroName = PageRequest.of(0, 10, Sort.by("realName"));
Page<Hero> heroesByRealName = heroRepository.findAllByRealName("Bruce", pageableRealName);
Page<Hero> heroesByHeroName = heroRepository.findAllByHeroName("Batman", pageableHeroName);
```
## Comparators

**Spring Data JPA** let us specify many types of queries, using one or more of the next comparators before the attribute name

* Containing
* Like
* IgnoreCase

``` java
Iterable<Hero> heroes = heroRepository.findAllByRealNameContaining(String name);
Iterable<Hero> heroes = heroRepository.findAllByRealNameLike(String name);
Iterable<Hero> heroes = heroRepository.findAllByRealNameIgnoreCase(String name);

```

Also we can combine those queries

``` java
Iterable<Hero> heroes = heroRepository.findAllByHeroNameContainingIgnoreCase(String heroName);
```

for example
``` java 
Iterable<Hero> heroes = repo.findAllByHeroNameContainingIgnoreCase("bat");
```
returns 
``` json
[
   {
      "realName":"Bruce Wayne",
      "heroName":"Batman"
   },
   {
      "realName":"Barbara Gordon",
      "heroName":"Batgirl"
   }
]
```

# Conclusion
So, with **Spring Data Repositories** we have a set of methods to retrieve our data from the database. This set of methods is more 
than enough for most of our porpouses.

**Spring Data JPA** let us write less code in order to make our JPA queries. The use of comparators is a clear 
example of how easy is implements queries to retrieve our data.

The use of comparators is nicer if you query is not to much complicated, but if you need more control or more specific queries or your query is complicated, you can use **@Query** annotation to specify custom queries.

Further information

https://spring.io/projects/spring-data-jpa

https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html
