---
type: exercise
acronym: bauzeicher20-cleancode-solid
title: Refactoring der "Bauzeichner 2.0"-Software aus dem Video nach SOLID und Clean Code 
navLabel: Refactoring nach SOLID und Clean Code
summary: >
    Das im Video zu den SOLID-Prinzipien vorgestellte Beispiel der "Bauzeichner 2.0"-Software wird in dieser Übung einmal gründlich
    gefactored - gemäß der Prinzipien von SOLID und Clean Code.
videos:
    - st2-102
    - st2-103
    - st2-104
    - st2-105
    - st2-106
    - st2-107
    - st2-108
dependsOnExercises:
durationMin: 300
withSolution: false
---


Alle nachfolgenden Schritte beziehen sich auf das Beispiel-Repo 
[Bauzeichner2.0-CleanCode-SOLID](https://gitlab.com/archi-lab/public/bauzeichner2.0-cleancode-solid). Das Repo ist
so aufgebaut, dass es **zwei Java-Projekte** enthält.  

* `before_refactoring` ist die Version, die DDD- und SOLID-Prinzipien verletzt. Alle entsprechenden Tests 
  sind rot (bzw. gelb, in IntelliJ). Funktional ist die Implementierung ok (wenn auch unvollständig und 
  sehr lieblos runtergehauen ...), was man daran sieht, dass die funktionalen Unit-Tests im
  Package `functionaltests` grün sind.
* `after_refactoring` enthält die Variante nach dem Refactoring.

**Hinweis**: Wegen der zwei Projekte sollten Sie also nicht den Top-Level-Ordner `bauzeichner2.0-cleancode-solid`
in IntelliJ öffnen, sondern die beiden unterliegenden Ordner `before_refactoring` und `after_refactoring` in
zwei verschiedenen IntelliJ-Instanzen.



## Schritt 1: Clean-Code-Prinzipien anwenden

Schauen Sie sich das Video an, und finden Sie Verstöße gegen die Clean-Code-Prinzipien. 
Das [PMD-Plugin für IntelliJ](https://www.archi-lab.io/pmd) kann Ihnen dabei helfen.

* Gehen Sie die Refactoring-Schritte durch. 
* Ich werde das ebenfalls im Live-Coding machen. 
* Anschließend schauen Sie Ihren eigenen Code aus Meilenstein M0 an. Welche Clean-Code-Prinzipien verletzen Sie dort?

{% if page.withSolution %}
(Die Lösung finden Sie - genau wie auch bei den nachfolgenden Schritten 2 ... 3c - in dem Projekt `after_refactoring`.)
{% endif %}


## Schritt 2: Package-Struktur und Namenskonventionen entsprechend DDD-Schichtenarchitektur 

Machen Sie gemäß unserer Konventionen für die [DDD-Schichtenarchitektur](../../material/st2/DDD-Schichtenarchitektur.pdf)
ein Refactoring des Übungs-Repos, so dass die DDD-Konventionen eingehalten werden. Denken Sie auch an die Namenskonventionen.



## Schritt 3a: SOLID-Prinzipien anwenden - Single Responsibility Principle

Finden Sie den hauptsächlichen Verstoß gegen das Single Responsibility Principle (SRP) im Code. Beheben Sie ihn.


## Schritt 3b: SOLID-Prinzipien anwenden - Open-Closed Principle

Finden Sie den hauptsächlichen Verstoß gegen das Open-Closed Principle (OCP) im Code. Beheben Sie ihn.


## Schritt 3c: SOLID-Prinzipien anwenden - Dependency Inversion Principle

Finden Sie den hauptsächlichen Verstoß gegen das Dependency Inversion Principle (DIP) im Code. Beheben Sie ihn.

{% if page.withSolution %}

### Lösung

Das allgemeine Vorgehen ist [auf einer eigenen Infoseite](../../infopages/coding/zykel-aufloesen-mit-dip.html)
beschrieben. Wenn Sie sich das Projekt `before_refactoring` anschauen, dann haben die Klassen `Canvas` und `DrawingElement` 
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
rücken darf). Diese gegenseitige Abhängigkeit wird man also nicht so leicht los. Nach Schritt 2
(Anlegen der passenden Package-Strukturen) und Schritt 3a (Aufbrechen von `DrawingElement` in
die beiden Klassen `Door` und `Window` sieht dieser Zykel so aus (`domain` und `application`-Packages
sind aus Übersichtlichkeitsgründen nicht gezeigt):  

![Zykel nach Schritt 3a](../../models/st/bauzeichner_after_step3a.png)


#### Abhilfe durch das Dependency Inversion Principle

Mit dem Dependency Inversion Principle kann man diese Abhängigkeiten aufheben. (Schauen Sie sich
bitte das entsprechende Video dazu an, siehe oben auf dieser Seite!) Dafür muss man auf der 
Seite des `Canvas`-Package Interfaces definieren, die sozusagen die "Erwartungen" der `Canvas`-Seite
ausdrücken.

* Man braucht ein Interface `Drawable`, das aus Sicht von `Canvas` alle Eigenschaften und nötigen 
  Methoden eines Zeichenelements enthält. 
* Zusätzlich braucht man noch ein Service-Interface `DrawableServiceInterface`, das 
  die Methoden für das Lifecycle-Management eines `Drawable` vorgibt. 
* Beide Interfaces werden dann tatsächlich auf der `DrawingElement`-Seite implementiert. 
  Die Implementieren referenzieren die Interfaces auf der `Canvas`-Seite.

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

{% endif %}
