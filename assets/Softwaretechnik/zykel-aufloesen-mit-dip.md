---
type: infopage
acronym: zykel-aufloesen-mit-dip
title: Zykel auflösen mittels Dependency Inversion Principle
navLabel: Zykel auflösen mittels Dependency Inversion Principle
summary: >
    Wenn man ein Domain-Driven Design mit Spring JPA umsetzt, helfen bestimmmte Patterns, die immer wieder
    verwendet werden können. Wenn Sie nicht so genau wissen, was mit "Patterns" gemeint ist, dann schauen Sie sich
    das Video dazu an. 
videos:
    - st2-101
    - st2-107 
    - st1-202
---

Zykel zwischen Packages (oder *Aggregates* in DDD-Begriffen) in Ihrem Code bedeuten eine enge Kopplung dieser 
Packages. Sie können sie nicht unabhängig voneinander testen oder wiederverwenden. Im allgemeinen bedeutet das
häufig auch, dass der Code unübersichtlich wird - Konzepte werden hin- und her-referenziert, und man verliert
schnell den Überblick, wenn man sich den Code neu anschaut.

Man kann also sagen, dass Zykel eine erhebliche **technische Schuld** darstellen. Daher sollte man Zykel so gut 
wie immer vermeiden. Dieser Artikel zeigt Ihnen, wie Sie das machen können.

## Allgemeines Vorgehen

Für das Auflösen von Zykeln können Sie sich an folgendem Vorgehen orientieren. 

![Vorgehen-Zykel](../../models/st/Vorgehen-Zykel.png)



### 1. Zykel zwischen Aggregates A und B entdeckt (A => B, B => A)

Im Praktikum nutzen wir das Tool [ArchUnit](https://www.archunit.org/), um gewisse Architektur-Regeln 
sicherzustellen - darunter auch die Zykelfreiheit. ArchUnit produziert leider sehr lange Fehlermeldungen, 
die man aber gut verstehen kann, wann man sich einmal an die richtige Stelle "gescrolled" hat. Wenn Sie einen
oder mehrere Zykel haben, dann werden Sie in der Fehlermeldung zum Zykeltest Passagen wie die folgende finden:

```
java.lang.AssertionError: 
Architecture Violation [Priority: MEDIUM] - Rule 'slices matching '..solution.(*)..' should be free of cycles' was violated (3 times):
Cycle detected: Slice order -> 
                Slice user -> 
                Slice order
  1. Dependencies of Slice order
    - Constructor <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.<init>(thkoeln.archilab.ecommerce.solution.order.application.OrderService, thkoeln.archilab.ecommerce.solution.user.application.UserService, thkoeln.archilab.ecommerce.solution.order.application.OrderDTOMapper)> has parameter of type <thkoeln.archilab.ecommerce.solution.user.application.UserService> in (OrderRestController.java:0)
    - Field <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.userService> has type <thkoeln.archilab.ecommerce.solution.user.application.UserService> in (OrderRestController.java:0)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.application.UserService.getUserRepo()> in (OrderRestController.java:42)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.domain.UserRepository.existsByMailAddressString(java.lang.String)> in (OrderRestController.java:42)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.application.UserService.getUserRepo()> in (OrderRestController.java:44)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.domain.UserRepository.findByMailAddressString(java.lang.String)> in (OrderRestController.java:44)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.application.UserService.getUserRepo()> in (OrderRestController.java:47)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.domain.User.getOrders()> in (OrderRestController.java:47)
    - Method <thkoeln.archilab.ecommerce.solution.order.application.OrderRestController.getAllOrders(java.lang.String, java.lang.String)> calls method <thkoeln.archilab.ecommerce.solution.user.domain.UserRepository.findByMailAddressString(java.lang.String)> in (OrderRestController.java:47)
  2. Dependencies of Slice user
    - Constructor <thkoeln.archilab.ecommerce.solution.user.domain.User.<init>(java.lang.String, java.util.UUID, thkoeln.archilab.ecommerce.domainprimitives.MailAddress, thkoeln.archilab.ecommerce.domainprimitives.PersonalAddress, java.util.List, thkoeln.archilab.ecommerce.solution.shoppingbasket.domain.ShoppingBasket, thkoeln.archilab.ecommerce.solution.delivery.domain.Delivery)> has generic parameter type <java.util.List<thkoeln.archilab.ecommerce.solution.order.domain.Order>> with type argument depending on <thkoeln.archilab.ecommerce.solution.order.domain.Order> in (User.java:0)
    - Field <thkoeln.archilab.ecommerce.solution.user.domain.User.orders> has generic type <java.util.List<thkoeln.archilab.ecommerce.solution.order.domain.Order>> with type argument depending on <thkoeln.archilab.ecommerce.solution.order.domain.Order> in (User.java:0)
    - Method <thkoeln.archilab.ecommerce.solution.user.domain.User.getOrders()> has generic return type <java.util.List<thkoeln.archilab.ecommerce.solution.order.domain.Order>> with type argument depending on <thkoeln.archilab.ecommerce.solution.order.domain.Order> in (User.java:0)
    - Method <thkoeln.archilab.ecommerce.solution.user.domain.UserRepository.getOrdersOfUser(thkoeln.archilab.ecommerce.domainprimitives.MailAddress)> has generic return type <java.util.List<thkoeln.archilab.ecommerce.solution.order.domain.Order>> with type argument depending on <thkoeln.archilab.ecommerce.solution.order.domain.Order> in (UserRepository.java:0)
    - Method <thkoeln.archilab.ecommerce.solution.user.domain.UserRepository.returnOrderOfUser(thkoeln.archilab.ecommerce.domainprimitives.MailAddress)> has generic return type <java.util.List<thkoeln.archilab.ecommerce.solution.order.domain.Order>> with type argument depending on <thkoeln.archilab.ecommerce.solution.order.domain.Order> in (UserRepository.java:0)
```

`Slice` ist der ArchUnit-Term für `Package`. Die Ausgabe sagt also, dass es eine zyklische Abhängigkeit zwischen `Order` und `User` 
gibt. Man kann auch noch weitere Informationen aus dem Ausschnitt entnehmen, dazu später mehr. 



### 2. Domain Model: Wie sollte die Abhängigkeit sein? (entweder A => B oder B  => A)

Wie sollte (nachdem wir fertig sind) die Abhängigkeit zwischen `Order` und `User` aussehen? Hierzu wendet man die 
Regel "Vom Speziellen zum Allgemeinen" an (siehe [Video dazu](https://www.youtube.com/watch?v=RufBXPTt1SA)).
Damit die nachfolgenden Beispiele auch visuell leichter nachzuvollziehen sind, färben wir die
nachfolgenden "allgemeinen" Elemente gelb ein und die "speziellen" grün.

![Abhängigkeit zwischen Allgemeinem und Speziellem](../../models/st/abhaengigkeit-allgemein-speziell.png)

In unserem Fall ist `User` ein allgemeineres Konzept als `Order`. Das heißt, dass `Order` von `User` abhängen sollte.

![Abhängigkeit zwischen Order und User](../../models/st/order-user-allgemein-speziell.png)



### 3. Zykel versehentlich oder unvermeidlich?

Nun wird geprüft, ob die gegenseitige Abhängigkeit zwischen `Order` und `User` fachlich nötig ist - z.B.  
wegen einem bestimmten fachlichen Feature. Das ist in diesem speziellen Fall nicht so. 


 
### 4. Refactoring des Codes ohne Dependency Inversion Principle

Damit wären wir in dem linken Zweig der obigen Grafik und damit bei Schritt 4: Refactoring des Codes, so dass strikt 
`Order` => `User` gilt, aber nicht mehr `User` => `Order`. Hierzu brauchen wir das Dependency Inversion Principle **NICHT**! Es genügt, wenn wir die Abhängigkeit `User` => `Order`
finden und eliminieren. Dazu hilft wieder die obige Ausgabe von ArchUnit (die Pfade habe ich der Übersichtlichkeit halber 
gekürzt):

```
(...)
  2. Dependencies of Slice user
    - ...
    - Field <User.orders> has generic type <java.util.List<Order>> with type argument depending on <Order> in (User.java:0)
    - Method <User.getOrders()> has generic return type <java.util.List<Order>> with type argument depending on <Order> in (User.java:0)
    - Method <UserRepository.getOrdersOfUser(MailAddress)> has generic return type <Order>> with type argument depending on <Order> in (UserRepository.java:0)
    - Method <UserRepository.returnOrderOfUser(MailAddress)> has generic return type List<Order>> with type argument depending on <Order> in (UserRepository.java:0)
(...)    
```

Also: `User` hat eine Liste von `Order`-Objekten. Das ist die Abhängigkeit, die wir im Code eliminieren müssen.




## Unvermeidbare zyklische Abhängigkeiten - Anwendung des Dependency Inversion Principle

Wenn wir aber im Fall "5. Zykel ist notwendig" landen, dann müssen wir das Dependency Inversion Principle anwenden.
Dies ist eine **immer funktionierende Methode**. Sie können sich zu diesem Prinzip das oben genannten Video
anschauen.


### Beispiel-Repo zum Nachvollziehen des Dependency Inversion Principle

Es gibt auch ein [Repo, das das Beispiel aus dem Video aufgreift](https://gitlab.com/archi-lab/public/bauzeichner2.0-cleancode-solid).
Wir haben dieses Repo gemeinsam in einem der ST2-Workshops bearbeitet. In dem `after_refactoring`-Teilprojekt dieses Repos
sehen Sie, wie Sie die Interfaces und Implementierungen wählen müssen, um die Abhängigkeiten aufzulösen. Dieses Beispiel 
stellen wir hier Schritt für Schritt vor. Vielleicht schauen Sie einmal in das Repo hinein und verstehen, worum es geht. 


### Noch einmal Schritt 2 für das Bauzeichner-Beispiel: Wie sollte die Abhängigkeit sein?

Machen wir noch einmal Schritt 2 aus dem obigen Vorgehen: Wie sollte die Abhängigkeit zwischen `Canvas` und `DrawingElement`
aussehen? Nach unserer Regel ist `Canvas` ein allgemeineres Konzept als `DrawingElement`. Das heißt, dass die Abhängigkeit
`DrawingElement` => `Canvas` sein sollte, aber nicht umgekehrt. (Wenn `Canvas` keine Kenntnis von speziellen `DrawingElements`
hat, dann hat das den zusätzlichen Vorteil, dass man später neue `DrawingElements` hinzufügen kann, ohne den `Canvas` zu ändern.) 
Idealerweise sollte die Abhängigkeit zwischen `DrawingElement` und `Canvas` also so aussehen:

![Abhängigkeit zwischen Canvas und DrawingElement](../../models/st/canvas-drawingelement-allgemein-speziell.png)


### Noch einmal Schritt 3 für das Bauzeichner-Beispiel: Zykel versehentlich oder unvermeidlich?

Hier gibt es eine zyklische Abhängigkeit zwischen `Canvas` und `DrawingElement`. Machen wir hier noch einmal den schnellen 
Test aus **Schritt 3** oben: Ist die gegenseitige Abhängigkeit fachlich nötig? 
- Ein `Canvas` besteht aus mehreren `DrawingElements` und muss diese "kennen", um initial zu platzieren, in Gruppen zu
  verschieben, etc.
- Ein `DrawingElement` will sich selbst in der Größe skalieren können. Dazu muss es benachbarte `DrawingElements` kennen, 
  um Kollisionen zu vermeiden. Die bekommt es nur, in dem es den `Canvas` kennt.

Das heißt, der Zykel ist kein Versehen, sondern hat eine fachliche Ursache. Wenn Sie sich das Projekt 
`before_refactoring` anschauen, dann haben die Klassen `Canvas` und `DrawingElement`
die folgende zyklische Abhängigkeit:

![Zykel vorher](../../models/st/bauzeichner_before.png)

Die Abhängigkeiten sind auch beide tatsächlich nötig, denn `Canvas` muss ausgeben können, welche Elemente
es enthält, bzw. welche Nachbar-Elemente ein `DrawingElement` hat. Sonst kann sich das Element nicht
z.B. in der Größe skalieren, denn es darf ja nicht mit Nachbarn kollidieren. Dafür gibt es die Methode

``` java
    public List<DrawingElement> getNeighboursOf( DrawingElement drawingElement ) {
        // ...
    }
```    

in `Canvas`. Auf der anderen Seite muss `DrawingElement` seine Zeichenfläche kennen, sonst kann es
keine Move-Befehle ausführen (es wüsste ja sonst nicht, wie weit es z.B. nach links oder rechts
rücken darf). Diese gegenseitige Abhängigkeit wird man also nicht so leicht los. Nach 
Anlegen der passenden Package-Strukturen) und Aufbrechen von `DrawingElement` in
die beiden Klassen `Door` und `Window` sieht dieser Zykel so aus (`domain` und `application`-Packages
sind aus Übersichtlichkeitsgründen nicht gezeigt):

![Zykel nach Schritt 3a](../../models/st/bauzeichner_after_step3a.png)


## Schritt 5: Dependency Inversion Principle anwenden

Mit dem Dependency Inversion Principle kann man diese Abhängigkeiten aufheben. Schauen Sie sich
bitte das entsprechende Video dazu an, siehe oben auf dieser Seite!). Dies gliedert sich in
eine Anzahl von Teilschritten, die nachfolgend beschrieben sind. 


### Schritt 5a: "Erwartungs"-Interfaces auf der "allgemeinen" Seite definieren


Dafür muss man auf der
Seite des `Canvas`-Package Interfaces definieren, die sozusagen die "Erwartungen" der `Canvas`-Seite
ausdrücken.

* Man braucht ein Interface `Drawable`, das aus Sicht von `Canvas` alle Eigenschaften und nötigen
  Methoden eines Zeichenelements enthält.
* Zusätzlich braucht man noch ein Service-Interface `DrawableServiceInterface`, das
  die Methoden für das Lifecycle-Management eines `Drawable` vorgibt.

Das `Drawable`-Interface sieht dann so aus:
``` java
public interface Drawable {
    public abstract void setCanvas( Canvas canvas );

    public abstract Integer xBottomLeft();
    public abstract void setXBottomLeft( Integer xBottomLeft );
    public abstract Integer yBottomLeft();

    public abstract Integer width();
    public abstract Integer height();
}
```

Das `DrawableServiceInterface` ist die "erwartete" Funktionalität, um ein `Drawable` zu instanziieren,
zu speichern, zu laden und zu löschen. Es sieht (in einer ersten vereinfachten Version) so aus:
``` java
public interface DrawableServiceInterface {
    public void save( Drawable drawable );

    public Drawable createDefaultDoor();
}
```

#### Häufiges Anti-Pattern: Sinnlose Namen für die "Erwartungs"-Interfaces

Bemühen Sie sich, einen guten, passenden Namen für diese "Erwartungs"-Interfaces zu finden. In unserem Fall
kann man in einem Fall gut das Namenspattern `...able` verwenden, im anderen Fall passt das nicht so gut.
Der Name sollte aber immer ausdrücken, was die "Erwartung" der "allgemeinen" Seite an verschiedenen, 
nicht im einzelnen bekannten "speziellen" Elemente ist.
**Das DIP ist kompliziert genug - mit schlechten Namen wird es völlig undurchsichtig!**



### Schritt 5b: Interfaces auf der "allgemeinen" Seite im Code verwenden


Verwenden Sie die Interfaces auf der "allgemeinen" Seite im Code. Das heißt, die "grünen" Klassen alle
eliminiert werden, und Sie stattdessen nur die noch die Interfaces verwenden. Wenn es noch Fehler
gibt, dann waren Ihre Interfaces unvollständig - Sie brauchen dann ggfs. noch weitere Methoden darin.


### Schritt 5c: "Erwartungs"-Interfaces auf der "speziellen" Seite implementieren


Beide Interfaces werden dann tatsächlich auf der `DrawingElement`-Seite implementiert.
Die Implementierungen referenzieren die Interface-Definitionen auf der `Canvas`-Seite.

Damit ergibt sich folgendes Bild (die neuen Klassen sind rot eingefärbt). Am besten schauen Sie sich
das einmal im Code an. Die Abhängigkeiten sind dadurch tatsächlich **nicht mehr zyklisch**.
Jetzt kennt das `DrawingElement`-Package zwar `Canvas`, aber nicht mehr umgekehrt.

![Zykel nach Schritt 3c](../../models/st/bauzeichner_after_step3c.png)



#### Kleine Abwandlung aufgrund der Randbedingungen von Spring JPA: Abstrakte Klasse statt Interface

Wie Sie im Code sehen, entspricht die Implementierung nicht 100% der obigen Skizze. `Drawable`
ist tatsächlich eine abstrakte Klasse, kein Interface. Das ist deswegen nötig, weil Spring JPA
keine Interfaces via `@OneToMany` referenzieren kann, wohl aber abstrakte Klassen. Am Prinzip
ändert dieses Detail aber nichts. Die korrekte Modellierung ist dann so:

![Zykel nach Schritt 3c mit abstrakter Klasse](../../models/st/bauzeichner_after_step3c_abstractclass.png)


