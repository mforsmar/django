___Starta projekt___:
  django-admin startproject mysite

Mappstruktur som samtliga Djangoprojekt följer:
mysite/             <-- Yttre mapp, kan heta vad som helst
    manage.py       <-- Command-line utility, för att cmd-interagera med projektet
    mysite/         <-- Inre mapp, dess namn används för att importera saker "innanför", typ "mysite-urls"
        __init__.py <-- Tom fil, signalerar att denna mapp ska behandlas som ett Python package
        settings.py <-- Inställningar för projektet
        urls.py     <-- "URL-declaration" för projektet, "table of content" för hemsidan
        wsgi.py     <-- Entry-point för wsgi-servrar

Köra develope-servern (inte för live):
python manage.py runserver
- Läser in pythonkod konstant vid varje request --> Server lägger av vid syntaxfel

Dyker upp på  http://127.0.0.1:8000/, för att ändra port:
python manage.py runserver 8080

Django använder SQLite, för andra databaser så behöver de database bindings fixas

I settings.py:
INSTALLED_APPS innehåller de Django-applications som är aktiva i Django-instansen
- django.contrib.admin --> Adminsidan.
- ^.auth               --> Authenticationsystemet.
- ^.contenttypes       --> Session framework.
- ^.messages           --> Messaging framework.
- ^.staticfiles        --> Framework för att hantera static files.

Skapa tabeller i databasen utefter innehållet i settings.py och INSTALLED_APPS:
python manage.py migrate

Skapa app:
python manage.py startapp polls

Appen innehåller datamodeller som bestämmer datans behaviour, som finns uppstyltade i models.py
I en poll är två lämpliga datamodeller Question samt Choice, som är egna klasser och subclassar  django.db.models.Model.
Varje datamodell har ett gäng klassvariabler som var för sig representerar ett databasfält.
Dessa variabler har som fields med argument, t.ex. models.CharField(max_length=200). Man kan även sätta default-argument.
Med dessa kan Django skapa ett databas-schema och database-access-API för questions&choice.

Berätta för projektet att 'Polls' är installerat:
- Läggar till "polls" i INSTALLED_APPS
- python manage.py makemigrations polls
- makemigrations berättar för Django att datamodeller uppdaterats och ändringarna ska sparas som en migration.

Se vad för SQL-kod som en migrations (här "polls") innebär:
python manage.py sqlmigrate polls 0001
Migrations gör att ens databas kan ändras under tiden utan att manuellt behöva ta bort/lägga till nya tabeller.

Sammanfattning: Hur man ändrar datamodeller.
- Change your models (in models.py).
- Run python manage.py makemigrations to create migrations for those changes
- Run python manage.py migrate to apply those changes to the database.

I Databas-access-API:n:
- För att dessa de commands som sedan kommer att fylla och fråga databasen.
- Starta interactive shell: python manage.py shell
- Se alla Question-object: Question.objects.all()
Skapa ny fråga, spara och sen titta på frågans id, måste även importera tidszonen.
- from django.utils import timezone
- q = Question(question_text="What's new?", pubdate=timezone.now())
- q.save()
- q.id
- q.question_text
- q.pub_date
Ändra och spara fråga:
- q.question_text = "What's up?"
- q.save

En "toString()" i datamodellklasser:  __str__()
Man kan lägga till egna metoder till datamodeller.

Fler commands:
- Question.objects.filter(id=1)
- Question.objects.filter(question_text__startswith='What')
- c.delete()

Skapa användare som kan komma åt adminsidan:
- python manage.py createsuperuser
- Detta kan sedan nås genom lokala domänen /admin/
- "Groups" och "Users" som man ser listade på /admin/ är provided av django.contrib.auth
- För att se sin app så måste den importas och sen registreras i polls/admin.py
- from polls.models import Question
  admin.site.register(Question)
-
