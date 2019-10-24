# Spring Data JPA

When we develop a Rest API maybe we have a lot of information to show or share. An important consideration is how we can show 
those data efficiently. A way to improve how we show data is paging those data, in this way the client (web explorer, mobile app)
can consume those data in small parts.

In this post I show how can we do paging and sorting data with one of the hottest technologies today 
[Spring Data JPA](https://spring.io/projects/spring-data). The purpose of **Spring Data** according to his page is:

*"provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.\
It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies."*

## Configuration
First, we need to create an **Entity** to have a model of our data

``` java
@Entity(name="heroe")
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

To be able to share the data, we need to recover the data first, so, we need to create a repository 
(*HeroRepository*)

``` java
public interface HeroRepo extends PagingAndSortingRepository<Hero, Long> {
}
```
Having extended from [PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)
we have the following methods (without coding anything)

``` java
List<Hero> findAll(Pageable pageable);
List<Hero> findAll(Sort sort);
```
## Paging and Sorting
After having created our repository, which implement *PagingAndSortingRepository* interface, we can recover our data
as follow

``` java
Pageable pageable = PageRequest.of(0, 20);
Page<Hero> heroes = heroRepository.findAll(pageable);
```

First argument of **PageRequest** is the page number and the second is the size of the set. How we can see, 
*heroRepository.findAll(pageable);* returns a *Page\<T>* instance. *Page\<T>* instance contains the 
list of heroes and the total of avalaible pages.

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
We can combine paging and sorting too.
``` java
Pageable pageable = PageRequest.of(0, 10, Sort.by("heroName"));
Page<Hero> heroes = heroRepository.findAll(pageable);
```

## Custom Paging and Sorting

Si bien **Spring Data** nos proporciona una buena cantidad de métodos para obetener los datos, algunas veces no es suficiente. Así que podemos crear las firmas de nuestros métodos dependiendo de qué datos queramos. Pr ejemplo, si quisiéramos obtener la lista de todos los super héroes con cierto nombre, tendríamos un método con la siguiente firma

``` java
List<Hero> heroes = heroRepository.findAllByRealName(String name, Pageable pageable);
List<Hero> heroes = heroRepository.findAllByHeroName(String heroName, Pageable pageable);
```
A pesar de ser un método especial que nosotros queremos **Spring Data** conoce las firma y sabe que después de *findAllBy* viene a continuación un atributo de la clase. Una vez haber creado la firma del método, podemos utilizarlo de manera similar a como lo vimos anteriormente

``` java
Pageable pageableRealName = PageRequest.of(0, 10, Sort.by("heroName"));
Pageable pageableHeroName = PageRequest.of(0, 10, Sort.by("realName"));
Page<Hero> heroesByRealName = heroRepository.findAllByRealName("Bruce", pageableRealName);
Page<Hero> heroesByHeroName = heroRepository.findAllByHeroName("Batman", pageableHeroName);
```
## Otros operadores de comparación
**Spring Data**  te permite especificar diferentes tipos de comparadores, esto, usando una de las siguientes *keywords* seguido del nombre del atributo del **Entity**

* Containing
* Like
* IgnoreCase


``` java
List<Hero> heroes = heroRepository.findAllByRealNameContaining(String name);
List<Hero> heroes = heroRepository.findAllByRealNameLike(String name);
List<Hero> heroes = heroRepository.findAllByRealNameIgnoreCase(String name);
```

Tambien se pueden combinar estas busquedas utilizando el *keyword* **And**

``` java
List<Hero> heroes = heroRepository.findAllByRealNameContainingAndIgnoreCase(String name);
```

De esta manera tenemos un conjunto de métodos muchas veces suficiente para poder recuperar los datos de la BD.

# Conclusión
**Spring Data JPA** nos ofrece bastantes features que nos permiten trabajar con JPA de una manera más fácil, el uso de los comparadores es un claro ejemplo de la facilidad para implementar búsquedas en nuestros datos. Es claro que si necesitamos búsquedas más complejas tal vez el escribir las queries de forma nativa es lo más conveniente.


https://www.baeldung.com/spring-data-jpa-pagination-sorting

https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html

https://thoughts-on-java.org/ultimate-guide-derived-queries-with-spring-data-jpa/

