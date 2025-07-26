---
type: infopage
acronym: adapter-pattern
title: Adapter-Pattern (a.k.a. Anti-Corruption-Layer)
navLabel: Adapter-Pattern (a.k.a. Anti-Corruption-Layer)
summary: >
    Wenn man ein Domain-Driven Design mit Spring JPA umsetzt, helfen bestimmmte Patterns, die immer wieder
    verwendet werden können. Wenn Sie nicht so genau wissen, was mit "Patterns" gemeint ist, dann schauen Sie sich
    das Video dazu an. 
videos:
    - st2-101
---


In den Praktikumsaufgaben gebe ich Ihnen Interfaces vor, die Sie nicht ändern dürfen. Das ist
durchaus absolut praxisnah! Nehmen Sie an, Sie bauen ein echtes ECommerce-Portal. Da werden
Sie haufenweise APIs bekommen, die Sie nicht ändern können - z.B. von Paypal, von DHL, ...
Vielleicht gefällt Ihnen der Zuschnitt der APIs nicht (zu groß, zu klein, ...),
vielleicht verwenden die andere Datenstrukturen als Ihr eigener Code, etc. 
Jammern hilft nicht, Sie müssen diese (API-)Interfaces so nehmen, wie sie kommen.

## Bauen eines "Anti-Corruption-Layer" mittels Adapter-Pattern 

Das heißt aber ausdrücklich **nicht**, dass Sie die Datenstrukturen aus diesen fremden
Interface auch in den eigenen Code "durchschleifen" müssen! Bauen Sie einfach einen
sogenannten _Anti-Corruption-Layer_ mit Hilfe des **Adapter-Patterns**.

![Adapter-Pattern für einen Anti-Corruption-Layer](../../models/st/Anti-Corruption-Layer.png)

Das gelbe Interface kommt von außen, mit einer Struktur, die Ihnen nicht gefällt. Die 
grünen Klassen sind von Ihnen selbst. Sie implementieren das gelbe Interface mit einem
`AdapterService`. Der übersetzt dann die gelbe, fremde Struktur in Ihre eigenen, "grünen"
Strukturen, indem er Ihre eigenen Services aufruft. Damit haben Sie die unerwünschten 
fremden Strukturen aus Ihrem eigenen Code "weggeblockt". So einfach ist das :-). 


## Wo sollten Adapter in Ihrem Code liegen?

Gemäß der [Schichtenarchitektur nach Eric Evans](../../infopages/ddd/package-convention-ddd.html#abbildung-einer-schichtenarchitektur-innerhalb-jedes-aggregates), die 
wir in ST2 verwenden, sollten Adapter im `application`-Layer des passen Aggregates liegen. Unter dem Link
im vorigen Satz ist das nochmal genauer erklärt.
