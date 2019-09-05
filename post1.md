# Spring Data JPA
## Paginación

Cuando se desarrolla un API Rest y se tiene una gran cantidad de datos para mostrar una cosa importante qué realizar es la
paginación de esos datos, así el cliente (navegador web, app móvil) puede consumir esos datos en pequeñas cantidades. 

En esta entrada daremos un repaso a la paginación utilizando [Spring Data JPA](https://spring.io/projects/spring-data)

## Configuración
Lo primero que debemos tener es una Entity que modele el dato que queremos mostrar.

``` java
@Entity
public class Hero {
    @Id
    private long idHero;

    private String heroName;
    private String realName;
    private String superPower;

    // Constructors, getters and setters
}
```

## Repositorio Spring Data

Para poder compartir los datos en el API Rest lo que necesitamos es crear un *HeroRepository*

``` java
public interface HeroRepo extends PagingAndSortingRepository<Hero, Long> {
}
```

Al haber extendido de [PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)
tenemos el método 

``` java
List<Hero> findAll(Pageable pageable);
```

https://www.baeldung.com/spring-data-jpa-pagination-sorting
https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html