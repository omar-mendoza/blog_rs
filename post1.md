# Spring Data JPA
## Paginación y Ordenamiento

Cuando se desarrolla un API Rest y se tiene una gran cantidad de datos para mostrar una cosa importante qué realizar es la paginación de esos datos, así el cliente (navegador web, app móvil) puede consumir esos datos en pequeñas cantidades. 

En esta entrada daremos un repaso a la paginación y ordenamiento de los datos utilizando [Spring Data JPA](https://spring.io/projects/spring-data).La misión de **Spring Data** es, acorde a su página:

*"provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.\
It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies."*

## Configuración
Lo primero que debemos tener es una Entity que modele el dato que queremos mostrar.

``` java
@Entity
public class Hero {
    @Id
    private long idHero;

    private String heroName;
    private String realName;
    private List<String> superPowers;

    // Constructors, getters and setters
}
```

## Repositorio Spring Data

Para poder compartir los datos con el API Rest lo que necesitamos es crear un *HeroRepository*

``` java
public interface HeroRepo extends PagingAndSortingRepository<Hero, Long> {
}
```

Al haber extendido de [PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)
tenemos los siguientes métodos

``` java
List<Hero> findAll(Pageable pageable);
List<Hero> findAll(Sort sort);
```
practicamente sin haber programado algún tipo de lógica.

## Paginación y Ordenación
Una vez que tenemos nuestro repositorio, el cual implementó la interfaz *PagingAndSortingRepository* podemos mandar a llamar los datos de manera paginada
``` java
Pageable pageable = PageRequest.of(0, 10);
Page<Hero> heroes = heroRepository.findAll(pageable);
```
Como se ve, esta consulta devuelve un *Page\<T>*. Además de tener la lista de héroes, la instancia *Page\<T>* conoce el total de número de páginas disponibles.

De manera similar, podemos hacer una ordenación de manera concisa
``` java
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName"));
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName").ascending());
Page<Hero> heroes = heroRepository.findAll(Sort.by("heroName").descending());
```
De igual manera podemos combinar **Paginación** y **Ordenamiento**

``` java
Pageable pageable = PageRequest.of(0, 10, Sort.by("heroName"));
Page<Hero> heroes = heroRepository.findAll(pageable);
```

## Paginación y Ordenación Personalizada

Si bien **Spring Data** nos proporciona una buena cantidad de métodos para obetener los datos, algunas veces no es suficiente. Así que podemos crear las firmas de nuestros métodos dependiendo de qué datos queramos. Pr ejemplo, si quisiéramos obtener la lista de todos los super héroes con cierto nombre, tendríamos un método con la siguiente firma en nuestro repositorio
``` java
public interface HeroRepository extends PagingAndSortingRepository<Hero, Integer> {

	public Iterable<Hero> findAllByHeroName(String heroName);
	public Iterable<Hero> findAllByHeroNameContaining(String heroName);
	public Iterable<Hero> findAllByHeroNameContainingIgnoreCase(String heroName);

}
```

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
De esta manera tenemos un conjunto de métodos muchas veces suficiente para poder recuperar los datos de la BD.

# Conclusión
**Spring Data JPA** nos ofrece bastantes features que nos permiten trabajar con JPA de una manera más fácil, el uso de los comparadores es un claro ejemplo de la facilidad para implementar búsquedas en nuestros datos. Es claro que si necesitamos búsquedas más complejas tal vez el escribir las queries de forma nativa es lo más conveniente.


https://www.baeldung.com/spring-data-jpa-pagination-sorting

https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html

https://thoughts-on-java.org/ultimate-guide-derived-queries-with-spring-data-jpa/

