# Des notes techniques

## POO

Abstraction, encapsulation, composition, héritage, polymorphisme, récursion ouverte (les méthodes d'objet peuvent appeler des méthodes du même objet)

Single responsibility : au maximum une raison de changer une classe  
Open-closed : ouverture aux extensions, mais fermeture aux modifications  
Liskov substitution : un objet de classe T peut être remplacé par un objet d'une classe S dérivée de T  
Interface segregation : le code ne doit pas être forcé de dépendre de méthodes qu'il n'utilise pas (=> faire de petites interfaces)  
Dependency inversion : dépendances envers des abstractions

## Programmation fonctionnelle

Fonctions de première classe, d'ordre supérieur, pures, récursivité, évaluation stricte ou non, typage, transparence référentielle (les variables sont constantes), structures de données

## Programmation événementielle

Exécution d'une fonction en réaction à un événement, généralement grâce à une boucle qui attend les événements

## Programmation orientée aspect

Ajout de comportements à un code existant. Cela permet de séparer les logiques

## Programmation réactive

Propagation du changement  
Optimise le temps CPU, lequel est coûteux

## Java

JVM : machine virtuelle exécutant le bytecode Java  
JRE : environnement d'exécution Java (JVM + lib)  
JDK : des libs et des outils dont le compilateur

Cycle de vie d'un Thread :  
NEW, avant l'appel à ``start()``  
RUNNABLE, en cours d'exécution ou prêt à l'être  
BLOCKED, en attente de moniteur  
WAITING, en attente d'un autre thread  
TIMED_WAITING, en attente d'un autre thread avec un temps maximal  
TERMINATED, l'exécution est terminée, normalement ou non

Une interface fonctionnelle n'a qu'une seule méthode abstraite (en dehors de celles d'``Object``)

### Java 17

```java
record Rectangle(double length, double width) { }
```

équivaut à

```java
public final class Rectangle {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    double length() { return this.length; }
    double width()  { return this.width; }

    public boolean equals...
    public int hashCode...

    public String toString() {...}
}
```

```java
public sealed interface Result<E, A> {

    record Success<E, A>(A value) implements Result<E, A> {
    }

    record Failure<E, A>(E error) implements Result<E, A> {
    }
}
```

```java
public sealed interface Shape permits Polygon { }
public non-sealed interface Polygon extends Shape { }
public final class UtahTeapot { }
public class Ring { }

public void work(Shape s) {
    UtahTeapot u = (UtahTeapot) s;  // Error
    Ring r = (Ring) s;              // Permitted
}
```

```java
public static boolean bigEnoughRect(Shape s) {
    if (!(s instanceof Rectangle r)) {
        // Cannot use r here
        return false;
    }
    return r.length() > 5; 
}
```

Une ``switch`` expression ``yield`` obligatoirement
```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> {
        System.out.println(6);
        yield 6;
    }
    case TUESDAY -> {
        System.out.println(7);
        yield 7;
    }
    case WEDNESDAY -> 3;
    default -> {
        throw new IllegalStateException("Invalid day: " + day);
    }
};
```

En preview, pour les ``switch`` expression et statement
```java
static void test(Object obj) {
    switch (obj) {
        case null -> System.out.println("null!");
        case Character c -> {
            if (c.charValue() == 7) {
                System.out.println("Ding!");
            }
            System.out.println("Character, value " + c.charValue());
        }
        case Integer i -> System.out.println("Integer: " + i);  
        default -> throw new IllegalStateException("Invalid argument"); 
    }
}
```

### Java 21

```java
static void printSum(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x+y);
    }
}
```

```java
public record Point(int x, int y) {
}

class Cartesian {

    static void printQuadrant(Point p) {
        switch (p) {
            case Point(var x, var y) when x > 0 && y > 0 -> 
                System.out.println("first");
            default -> 
                System.out.println("other");
        }
    }
}
```

Les virtual threads pour les applications concurrentes à haut débit.

### REST

Stateless, cachable, client-serveur, en couche, interface uniforme (identification des ressources, manipulations par des représentations, messages autodescriptifs, HATEOAS)

#### Pattern Saga

Une saga est une séquence de transactions locales. A la fin d'une transaction, un événement est publié pour lancer la transaction suivante.  
Si l'une rollback, des transactions de compensation sont lancées.
- Chorégraphie : chaque transaction publie des événements pour les autres services.
- Orchestration : l'orchestrateur indique aux services les transactions à exécuter.

### Spring

La ``BeanFactory`` fournit un mécanisme de configuration capable de gérer n'importe quel objet. L'``ApplicationContext`` ajoute une intégration à Spring AOP, l'internationalisation,
la publication d'événements et des contextes spécifiques.

#### Spring Boot

Facilite la création d'applications stand-alone de grade production basées sur Spring. Permet de commencer rapidement grâce à une vision "opinionnée" de Spring et de bibliothèques.
Peut embarquer un serveur, de la sécurité, des métriques, etc. La configuration par défaut se surcharge aisément.  
Les applications peuvent se lancer par un `jar` habituel ou un déploiement `war`.

#### Spring MVC

Composant de Spring Framework

#### Spring Data

Dérivation des requêtes à partir des noms de méthodes du `Repository`. Une classe annotée `@Repository` est éligible à la traduction des exceptions en `DataAccessException`.
Supporte l'audit (créateur, modificateur).

##### Spring Data JPA

`@Transactional` rollback, par défaut, sur une `RuntimeException` et une `Error`.  
Les isolations sont :
- READ_UNCOMMITTED : tous les phénomènes sont permis
- READ_COMMITTED : dirty read interdits
- REPEATABLE_READ : dirty read et non-repeatable reads interdits
- SERIALIZABLE : dirty read, non-repeatable reads et phantom reads interdits

Il est possible de préciser pour quels `Throwable` rollbacker ou non, de suggérer que la transaction est une lecture seule et de préciser un timeout.
La propagation peut être :
- MANDATORY : lève une exception si pas de transaction
- NESTED : crée une transaction dans une transaction, sinon crée une transaction
- NEVER : lève une exception si présence de transaction
- NOT_SUPPORTED : s'exécute hors transaction, suspend la transaction le cas échéant
- REQUIRED (défaut) : crée une transaction si besoin
- REQUIRES_NEW : crée une nouvelle transaction, suspend l'actuelle si besoin
- SUPPORTS : peu importe

#### Spring Security

La requête passe une `FilterChain` puis atteint la `DispatcherServlet`. Spring s'insère dans la FilterChain avec un `DelegatingFilterProxy` (fournie par Spring Framework).
Les filtres peuvent alors être des beans lazy-loadés (les filtres doivent être enregistrés avant).  
Spring Security fournit `FilterChainProxy` qui s'inscrit dans `DelegatingFilterProxy`. Ce proxy est configuré par une `SecurityFilterChain` (ou plusieurs, mais une seule reçoit la requête) et protège contre certaines attaques.  
A la différence des filtres Servlet, les filtres enregistrés dans une `SecurityFilterChain` peuvent être invoqués selon la `HttpServletRequest` et non seulement l'URL.
Parme les filtres standards, il y a ceux de l'authentification, de l'autorisation, etc.

#### Spring Batch

Un `Job` a un ou plusieurs `Step`, chacun ayant un `ItemReader`, un `ItemProcessor` et un `ItemWriter`. Un `Job` est lancé par un `JobLauncher` et les métadonnées sont dans un `JobRepository`.

### Quarkus

Proche de Spring Boot.  
En compilation native, l'exécutable a une moindre empreinte mémoire et un temps de démarrage plus court. La différence de performances est à mesurer à cause de l'absence de JIT.  
L'absence de JIT ne permet plus d'optimiser à l'exécution en fonction des statistiques d'utilisation des méthodes.

## Hibernate

@Entity  
@Id  
@Table  
@OneToMany et @ManyToOne  
@ManyToMany

## Angular

[Architecture en couche](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/)

Change Detection Strategy :  
- default : déclenché sur tout l'arbre sur événements, promesses, timers, changement de propriété ou manuellement
- OnPush : déclenché manuellement ou sur changement de références d'input, sur un événement du sous-arbre
  ou une émission du pipe ``async`` (car appelle la détection de changement)

Avec Zone.js, la détection de changements est exécutée automatiquement lorsque la file des microtasks - de la zone Angular - devient vide.

### Cycle de vie des composants

- ngOnInit : appelée une fois et après l'initialisation des inputs
- ngOnChanges : appelée si la détection de changements par défaut a repéré un changement
- ngDoCheck : il est possible de détecter et réagir à des changements non repérés par la détection par défaut (lancée avant cette fonction)
- ngAfterContentInit : appelée après l'initialisation du contenu projeté
- ngAfterContentChecked : appelée après la vérification du contenu projeté
- ngAfterViewInit : appelée après l'initialisation de la vue (le propre template du composant)
- ngAfterViewChecked : appelée après la vérification de la vue
- ngOnDestroy : appelée une fois, avant la destruction

La détection de changements par défaut repère les inégalités strictes des expressions utilisées dans le template.

### Injection de dépendances

En standalone, les services peuvent être fournis au niveau :
- de l'application. Le service est alors tree-shakeable et utilise la hiérarchie EnvironmentInjector.
- d'un composant. Le service est alors partagé par les différentes instances du composant et les composants/directives utilisés par celles-ci. Il n'est pas tree-shakeable et utilise la hiérarchie ElementInjector.
- de la configuration de l'application. Le service n'est alors pas tree-shakeable et utilise la hiérarchie EnvironmentInjector.

Le router est capable de créer des hiérarchies d'EnvironmentInjector. L'injecteur créé est alors lié à la route et ses enfants.

Par défaut, la résolution d'une dépendance cherche dans la hiérarchie Element puis dans Environment.

### Formulaires

#### Reactive forms

FormControl

#### Template-driven forms

ngModel

### Angular router

#### Nested routes

#### Lazy loading

#### Guards

## RxJS

## Vue

### Réactivité

### Vue Router
