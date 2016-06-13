### Sätta upp ett projekt
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

### Datamodeller
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

### Databasernas API
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

### Adminsidan
Skapa användare som kan komma åt adminsidan:
- python manage.py createsuperuser
- Detta kan sedan nås genom lokala domänen /admin/
- "Groups" och "Users" som man ser listade på /admin/ är provided av django.contrib.auth
- För att se sin app så måste den importas och sen registreras i polls/admin.py
- from polls.models import Question
  admin.site.register(Question)
På detta sätt så display:as de inladdade datamodellen. Model-field:sen, t.ex. "DateTimeField" och "CharField", matchar upp mot passande HTML input widget och vet hur det ska visas i Django admin. Field:sen får dessutom tillhörande JS-shortcuts, som "Today" och "Now".

För att customize:A admin-formulären så kan man skapa ett model admin object och skicka med det som parameter i register-anropet, se nedan.

    class QuestionAdmin(admin.ModelAdmin):

        # Enbart fields
        fields = ['pub_date', 'question_text']

        # Fieldset, med tupplar av ('field-titel', 'field')
        fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]
    admin.site.register(Question, QuestionAdmin)

Om man vill att objekt ska visas på ett annat objekts admin-sida så registrerar man det inte, utan låter objektet som ska ha en admin-sida bli en parent-model.

    inlines = [ChoiceInline]

##### Övriga customizes till adminsidan

För att lägga till kompletterande subrubriker till ett field, lägga till en ruta där användaren kan filtra de olika instanserna av datamodellen dess olika attribut samt fixa sökfält så kan nedanstående kod läggas till model-admin-objektet. De specifika filtreringsalternativen beror på vilken typ Model-field:sen är.

    list_display = ('question_text', 'pub_date')
    list_filter = ['pub_date']
    search_fields = ['question_text']

___Template___ : To be written.
### Views

- Är en "typ" av webbsida som uppfyller en specifik funktion eller har en specifik template.
- I Django är webbsidor och annat innehåll levererat genom views, där varje view representerar en Python-funktion. Dessa Python-funktioner skriv in i views.py. Django väljer view genom att kolla upp vilken URL som är efterfrågad (delen efter domänen i URL:en).
- För att jobba från en URL till en view så använder Django "URLconfs", som mappar URL-patterns (likt regex) till views.

När view:en ska mappas mot en URL så defineras urlpatterns i urls.py, där funktionen _url_ anropas. Denna funktion beskrivs mer utförligt längre ner. Därefter för att peka root-URLconf:en mot polls.urls i mysite/urls.py, där ett include()-anrop görs likt nedan.

    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', include(admin.site.urls)),

##### Url()-funktionen
Url-funktionen anropas med 4 argument, där de obligatoriska är *regex* och *view* och de valfria är *kwargs* och *name*.

###### Url()-argument: regex
Django kommer att börja kolla på det första regex:et och jämföra URL:en mot varje regex tills den matchar. Regex:et söker inte efter GET, POST eller domännamn. Exempel:
- I en request till "http://www.example.com/myapp/" söker URLconf efter "myapp/".
- I en request till "http://www.example.com/myapp/?page=3" letar URLconf efter "myapp/".

###### Url()-argument: view
När Django hittar en regex-match så ropas den spec:ade view-funktionen med ett _HttpRequest_-objekt som första parameter och de "fångade" värdena från andra parameters.

###### Url()-argument: kwargs
Godtyckliga keyword-argument kan skickas i dictionary till target view.

###### Url()-argument: name
Eventuellt namn på URLen så att denna kan refereras till på ett lättare sätt. På detta vis kan globala förändringar genomföras genom att ändra i en enda fil.

##### Konstruera views
Fler Python-metoder kan defineras i polls/views.py, och _urlpatterns_ kan utökas till flera patterns i polls/urls.py med lämpliga parametrar. När någon requestar en hemsida kommer Django att ladda in mysite.urls-Pythonmodulen då den pekas på genom ROOT_URLCONF-inställningen i settings.py. Den i sin tur hittar variabeln _urlpatterns_ och går därefter igenom de olika regex:arna i ordning. include()-funktionerna refererar till andra URLconfs. När Django stöter på include() så klipper den bort den delen av URLen som den dittills gått igenom och skickar med resten av URLen till den inkluderade URLconf:en för vidare processing.

Exempel på vad som händer om en användare går till en viss URL.
- Django kommer att matcha mot '^polls/'
- Sedan kommer Django klippa ur till den matchande texten "polls/" och skicka den resterande delen "34/" till 'olls.urls' URLconf för vidare processing. Där matchar den mot <span style="background-color:yellow">r'^(?P<question_id>\d+)/$'</span> vilket resulterar i ett anrop till detail()-funktionen i views.py:

      detail(request=<HttpRequest object>, question_id='34')

###### Konstruera mer funktionella views
