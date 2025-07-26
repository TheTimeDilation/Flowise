---
type: exercise
acronym: restImplementationInternship
title: REST-API für das "Internship Management System" implementieren
navLabel: REST-API mit Spring implementieren (Beispiel "Internship")
summary: In dieser Übung geht es darum, ein REST-API mit Hilfe von Spring Boot umzusetzen.
videos:
      - st2-306
      - st2-401
dependsOnExercises:
durationMin: 360
withSolution: true
display:
    language: de
    tocDepth: 2
---

Im Rahmen eines Campus-Management-Systems soll an der TH Köln auch ein **Internship Management System** (IMS) (eine 
Börse für Betriebspraktika bei Firmen) aufgebaut werden. Firmen bieten solche Praktika an, die dann von einer/m 
Student/in ausgefüllt werden können. 

Studierende müssen sich per Anwesenheitsnachweis (AttendanceCheck) die Tage bestätigen lassen, die sie wirklich bei 
der Firma waren (bei weniger als 60% Anwesenheit zählt das Praktikum nicht). Der Einfachheit halber zählen alle
Wochentage des Praktikums mit, es wird also für die 60%-Grenze nicht zwischen Wochenende und Arbeitstag unterschieden.

Zusätzlich können sich Studierende bei interessanten Firmen als "Follower" eintragen, so wie bei Twitter, um über
Praktikums-, Abschlussarbeits- und Jobangebote informiert zu werden. Hier sehen Sie das logische Datenmodell 
(domain model).

![Internship-Domain-Modell](../../models/st/Internship-Domain-Model.png)




## Ihre Aufgaben

Definieren Sie zuerst die Aggregates. Dann spezifizieren Sie ein REST API für einen Satz von Anforderungen. 
Schließlich implementieren Sie das API Schritt für Schritt.

Sie können auf einem existierenden Repo aufbauen. Das finden Sie hier: 
[https://git.archi-lab.io/students/exercises/internship/internship-rest-exercise](https://git.archi-lab.io/students/exercises/internship/internship-rest-exercise).
Dieses Repo enthält eine Anzahl von Unit Tests. Einige sind von vornherein schon grün - das sind die Tests, 
die die richtige Erzeugung der Sample-Daten und das Wenige an Geschäftslogik testen. Die anderen Tests sind für 
das REST API. Die machen Sie jetzt nach und nach grün :-). 

## Aufgabe 1: Bestimmen Sie die Aggregates

Definieren Sie die Aggregates. Sie können das z.B. einfach einzeichnen. 

{% if page.withSolution %}

### Lösung

![Internship-Aggregates](../../models/st/Internship-Aggregates.png)
 
{% endif %}


## Aufgabe 2: Definieren Sie das REST-API

Anmerkung: Die Aufgabenbeschreibung ist ab jetzt auf Englisch, weil das die Verwendung der richtigen Begriffe einfacher 
macht, und auch näher am Stil des Praktikums ist. 

The Internship Management System (IMS) allows to manage internships. Students and companies are used from other data 
sources. Therefore, the IMS doesn't allow to edit student or company data, or to enter new students / companies into
the IMS. In the implementation


In order to implement a web client for the IMS, the backend will have to offer at least the following 
REST API endpoints. 


#### REST1: Return all students currently managed in the system.
- For privacy reasons, a student's name and the matriculation number must never be displayed together.
- Therefore, this REST endpoint returns only a student's name and date of birth.
- The matriculation number must not be contained in the response.

#### REST1a: Register a new student, and return the student's matriculation number (2 endpoints)
- Send student data to register a new student (endpoint 1).
- Return the matriculation number of the new student (endpoint 2). The return data must not contain
  the student's name and date of birth - only the matriculation number.

#### REST2: Return a company currently managed in the system, identified by its ID
- The list of followers must not be returned. 
- Instead, the returned response must contain the total number of students following the company,
  as a field `numberOfFollowers`.
- There is a method in `Company` that calculates this number (not that this is very difficult :-).

#### REST3: Add a follower to a company
- Add a student as follower to a company

#### REST4: Enter a new internship
- Create a new internship.
- Student and company must be already known in the system.
- Return a response showing the internship.
- Student and company should be represented by their IDs, respectively.
- The list of attendance checks must not be returned. Instead, the current attendance percentage must be returned,
  as a field `attendancePercentage`.
- There is a method in `Internship` that calculates this percentage.

#### REST5: Get the internship for a given ID
- See also conditions in REST4 for the response object.

#### REST6: Correct the end date in an internship
- Change the given end date in the internship.
- See also conditions in REST4 for the response object.

#### REST7: Add an attendance check for an internship
- Add an attendance check to the internship

#### REST8: Clear all attendance checks for an internship
- Remove all attendance checks for the internship. 
- This needs to be done for privacy reasons at the end of the internship.

{% if page.withSolution %}

### Lösung

#### REST1: Return all students currently managed in the system.
- `GET /students`

#### REST1a: Register a new student, and return the student's matriculation number (2 endpoints)
- `POST /students`
- `GET /students/{student-id}/matrNumber`

#### REST2: Return a company currently managed in the system, identified by its ID
- `GET /companies/{company-id}`

#### REST3: Add a follower to a company
- `PUT /companies/{company-id}/followers/{student-id}`

#### REST4: Enter a new internship
- `POST /interships`

#### REST5:  Get the internship for a given ID
- `GET /interships/{intership-id}`

#### REST6: Correct the end date in an internship
- `PATCH /interships/{intership-id}`

#### REST7: Add an attendance check for an internship
- `POST /interships/{intership-id}/attendanceChecks`

#### REST8: Clear all attendance checks for an internship
- `DELETE /interships/{intership-id}/attendanceChecks`


{% endif %}

## Aufgabe 3: Implementieren Sie das REST API

Eine Beispiellösung finden Sie unter 
[https://git.archi-lab.io/students/exercises/internship/internship-rest-solution](https://git.archi-lab.io/students/exercises/internship/internship-rest-solution). 
Das Repo wird nach Ende der Übung sichtbar geschaltet. 


### Implementieren Sie REST1: Return all students currently managed in the system.

Sie können das API in den folgenden Schritten definieren (das gilt sinngemäß auch für alle nachfolgenden
API-Endpoints):
* Legen Sie ein Package `application` an. Alles, was zum API gehört, kommt da hinein.
* Legen Sie den REST-Controller an, wie im Video beschrieben. 
* Definieren Sie ein passendes DTO, das Sie für Request und Response gleichermaßen nutzen können. Nutzen Sie das 
  DTO im REST-Controller.
* Der REST-Controller sollte keine Business-Logik enthalten, weil er eine _technische_ Klasse ist. Also 
  legen Sie bitte einen ApplicationService an. Der macht dann alle Queries und Updates (und nutzt dafür dann
  wieder das Repository des eigenen Aggregates, sowie ApplicationServices der anderen Aggregates). 
* Ihr Application Service kann schon mit den DTOs arbeiten.
* Sie können den ModelMapper nutzen, um den Transfer von Entity zu DTO (und zurück) zu machen. Das kann in einer
  eigenen Mapperklasse sein, so wie in den Videos, oder aber auch einfach in den ApplicationService-Methoden.

Wenn Sie das korrekt implementieren, sollten die Tests aus  
`thkoeln.archilab.internship.student.StudentRESTGetAllTests` grün werden.


### Implementieren Sie REST2: Return a company currently managed in the system, identified by its ID

Zu beachten:
* Denken Sie bei REST1 über das passende DTO nach.

Wenn Sie das korrekt implementieren, sollten die Tests aus  
`thkoeln.archilab.internship.company.CompanyRESTGetTests` grün werden.


### Implementieren Sie REST3: Add a follower to a company

Zu beachten:
* Wenn Sie zweimal denselber Follower hinzufügen, sollte dieser nur 1x in der Beziehung auftreten. 

Danach sollten die Tests aus `thkoeln.archilab.internship.company.CompanyRESTAddFollowerTests` grün werden.


### Implementieren Sie REST4 (Enter a new internship) und REST5 (Get the internship for a given ID)

Implementieren Sie bitte beides zusammen, beide Aufrufe werden zusammen getestet. Beachten Sie die Randbedingungen! 
Danach sollten die Tests aus `thkoeln.archilab.internship.internship.InternshipRESTCreateTests` grün werden.


### Implementieren Sie REST6: Correct the end date in an internship

Hinweis: Wenn Sie den `ModelMapper` mit folgender Zeile konfigurieren, dann "überspringt" er Properties, die 
im DTO `null` sind - sehr praktisch für `PATCH`-Operationen. Wenn man das nicht macht, sucht man lange nach dem
Fehler :-(.

`modelMapper.getConfiguration().setSkipNullEnabled( true );`

Danach sollten die Tests aus `thkoeln.archilab.internship.internship.InternshipRESTPatchTests` grün werden.


### Implementieren Sie REST7 (Add an attendance check for an internship) und REST8: Clear all attendance checks for an internship

Implementieren Sie bitte beides zusammen, beide Aufrufe werden zusammen getestet.
Danach sollten die Tests aus `thkoeln.archilab.internship.internship.InternshipRESTPatchTests` grün werden.



