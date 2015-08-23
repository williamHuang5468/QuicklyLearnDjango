#Writing your first Django app

## Table

[#Part2](#part2)
[#Part3](#part3)

### Goal 

    we’ll walk you through the creation of a basic poll application.

it has two part.

- A public site that lets people view polls and vote in them.
- An admin site that lets you add, change and delete polls.

## 1. step - Create probject

command :

django-admin startproject `projectName`

ex.

>> django-admin startproject mysite

They will create

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

- The outer mysite/ root directory is just a container for your project. Its name doesn’t matter to Django; you can rename it to anything you like.
- manage.py: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about manage.py in django-admin and manage.py.
- The inner mysite/ directory is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. mysite.urls).
- mysite/__init__.py: An empty file that tells Python that this directory should be considered a Python package. (Read more about packages in the official Python docs if you’re a Python beginner.)
- mysite/settings.py: Settings/configuration for this Django project. Django settings will tell you all about how settings work.
- mysite/urls.py: The URL declarations for this Django project; a “table of contents” of your Django-powered site. You can read more about URLs in URL dispatcher.
- mysite/wsgi.py: An entry-point for WSGI-compatible web servers to serve your project. See How to deploy with WSGI for more details.

## 2. Step - Database setup

    python manage.py migrate

>>>> Why are you need migrate?

>>>> What happen if you do migrate?

The migrate command looks at the INSTALLED_APPS setting and creates any necessary database tables according to the database settings in your mysite/settings.py file and the database migrations shipped with the app (we’ll cover those later). 

## 3. Step - Run server

    python manage.py runserver

## 4. Step - Create Model

command 

    python manage.py startapp `AppName`

ex. 

    python manage.py startapp polls



polls/models.py

    from django.db import models


    class Question(models.Model):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')


    class Choice(models.Model):
        question = models.ForeignKey(Question)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

Picture

    Question
        text(Char)
        date(Time)

    Choice 
        question(Qestion)
        text(Char)
        votes(Integer)

>>>> What kind of type that i have?

Ref : [Model Doc](https://docs.djangoproject.com/en/1.8/ref/models/instances/#django.db.models.Model)

Modify setting

    mysite/setting.py

    INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    `'polls',`
    )

### Update

Command 

    python manage.py makemigrations appName

ex.
    python manage.py makemigrations polls

You will get message:

    Migrations for 'polls':
        0001_initial.py:
            - Create model Question
            - Create model Choice
            - Add field question to choice

run migrate again to create those model tables in your database:

    python manage.py migrate

###小節

- Change your models (in models.py).
- Run python manage.py makemigrations to create migrations for those changes
- Run python manage.py migrate to apply those changes to the database.

## Playing with the API

    Python manage.py shell

database api:

    >>> from polls.models import Question, Choice   # Import the model classes we just wrote.

    # No questions are in the system yet.
    >>> Question.objects.all()
    []

    # Create a new Question.
    # Support for time zones is enabled in the default settings file, so
    # Django expects a datetime with tzinfo for pub_date. Use timezone.now()
    # instead of datetime.datetime.now() and it will do the right thing.
    >>> from django.utils import timezone
    >>> q = Question(question_text="What's new?", pub_date=timezone.now())

    # Save the object into the database. You have to call save() explicitly.
    >>> q.save()

    # Now it has an ID. Note that this might say "1L" instead of "1", depending
    # on which database you're using. That's no biggie; it just means your
    # database backend prefers to return integers as Python long integer
    # objects.
    >>> q.id
    1

    # Access model field values via Python attributes.
    >>> q.question_text
    "What's new?"
    >>> q.pub_date
    datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

    # Change values by changing the attributes, then calling save().
    >>> q.question_text = "What's up?"
    >>> q.save()

    # objects.all() displays all the questions in the database.
    >>> Question.objects.all()
    [<Question: Question object>]

Stop , this has `[<Question: Question object>]`, it need be text.

>>>> Notice : if you run this shell has some problem.

>>>> Delete the sqlite file, and rerun the command start by `python manager.py makemigrations appName` and `python manager.py migrate`

### Convert text

polls/models.py

    from django.db import models

    class Question(models.Model):
        # ...
        def __str__(self):              # __unicode__ on Python 2
            return self.question_text

    class Choice(models.Model):
        # ...
        def __str__(self):              # __unicode__ on Python 2
            return self.choice_text

### add new function

polls/models.py

    import datetime

    from django.db import models
    from django.utils import timezone


    class Question(models.Model):
        # ...
        def was_published_recently(self):
            return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

### Second Parts - Database API

    >>> from polls.models import Question, Choice

    # Make sure our __str__() addition worked.
    >>> Question.objects.all()
    [<Question: What's up?>]

    # Django provides a rich database lookup API that's entirely driven by
    # keyword arguments.
    >>> Question.objects.filter(id=1)
    [<Question: What's up?>]
    >>> Question.objects.filter(question_text__startswith='What')
    [<Question: What's up?>]

    # Get the question that was published this year.
    >>> from django.utils import timezone
    >>> current_year = timezone.now().year
    >>> Question.objects.get(pub_date__year=current_year)
    <Question: What's up?>

    # Request an ID that doesn't exist, this will raise an exception.
    >>> Question.objects.get(id=2)
    Traceback (most recent call last):
        ...
    DoesNotExist: Question matching query does not exist.

    # Lookup by a primary key is the most common case, so Django provides a
    # shortcut for primary-key exact lookups.
    # The following is identical to Question.objects.get(id=1).
    >>> Question.objects.get(pk=1)
    <Question: What's up?>

    # Make sure our custom method worked.
    >>> q = Question.objects.get(pk=1)
    >>> q.was_published_recently()
    True

    # Give the Question a couple of Choices. The create call constructs a new
    # Choice object, does the INSERT statement, adds the choice to the set
    # of available choices and returns the new Choice object. Django creates
    # a set to hold the "other side" of a ForeignKey relation
    # (e.g. a question's choice) which can be accessed via the API.
    >>> q = Question.objects.get(pk=1)

    # Display any choices from the related object set -- none so far.
    >>> q.choice_set.all()
    []

    # Create three choices.
    >>> q.choice_set.create(choice_text='Not much', votes=0)
    <Choice: Not much>
    >>> q.choice_set.create(choice_text='The sky', votes=0)
    <Choice: The sky>
    >>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

    # Choice objects have API access to their related Question objects.
    >>> c.question
    <Question: What's up?>

    # And vice versa: Question objects get access to Choice objects.
    >>> q.choice_set.all()
    [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
    >>> q.choice_set.count()
    3

    # The API automatically follows relationships as far as you need.
    # Use double underscores to separate relationships.
    # This works as many levels deep as you want; there's no limit.
    # Find all Choices for any question whose pub_date is in this year
    # (reusing the 'current_year' variable we created above).
    >>> Choice.objects.filter(question__pub_date__year=current_year)
    [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]

    # Let's delete one of the choices. Use delete() for that.
    >>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
    >>> c.delete()

# Part2

>> Django’s automatically-generated admin site

## Creating an admin user

create user

    python manage.py createsuperuser

## Start the development server

    python manage.py runserver

    http://127.0.0.1:8000/admin/

## Make the poll app modifiable in the admin

>> Add Field into admin page.

polls/admin.py

    from django.contrib import admin

    from .models import Question

    admin.site.register(Question)

## Customize the admin form

>> You can design your expect order.

polls/admin.py

    from django.contrib import admin

    from .models import Question


    class QuestionAdmin(admin.ModelAdmin):
        fields = ['pub_date', 'question_text']

    admin.site.register(Question, QuestionAdmin)

### fields change order

    class QuestionAdmin(admin.ModelAdmin):
        fieldsets = [
            (None,               {'fields': ['question_text']}),
            ('Date information', {'fields': ['pub_date']}),
        ]

None and 'Date information' is field name.

### show and hide dataset

    class QuestionAdmin(admin.ModelAdmin):
        fieldsets = [
            (None,               {'fields': ['question_text']}),
            ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
        ]

`{'fields': ['pub_date'], 'classes': ['collapse']})` is value.

The field can collapse is use `'classes': ['collapse']`.

## Adding related objects

Add Choice

admin.py

    from .models import Choice, Question

    admin.site.register(Choice)

Add Choice into Question

    from django.contrib import admin

    from .models import Choice, Question


    class ChoiceInline(admin.StackedInline):
        model = Choice
        extra = 3


    class QuestionAdmin(admin.ModelAdmin):
        fieldsets = [
            (None,               {'fields': ['question_text']}),
            ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
        ]
        inlines = [ChoiceInline]

    admin.site.register(Question, QuestionAdmin)

Change component form

    ChoiceInline(admin.TabularInline)


StackedInline

    Item
        1
        2
        3
    Item
        1
        2
        3

    Item
        1
        2

TabularInline

    Item
        1   2
    Item
        1   2
    Item
        1   2

## Customize the admin change list

    class QuestionAdmin(admin.ModelAdmin):
        # ...
        list_display = ('question_text', 'pub_date')

polls/admin.py

    list_filter = ['pub_date']

# Part3

>> We’re continuing the Web-poll application and will focus on creating the public interface – “views.”

## Philosophy

A view is a "type" of Web page in your Django application that serves a function and has a tempalte.

Ex. In a blog application, you might have the following views:

- Blog homepage – displays the latest few entries.
- Entry “detail” page – permalink page for a single entry.
- Year-based archive page – displays all months with entries in the given year.
- Month-based archive page – displays all days with entries in the given month.
- Day-based archive page – displays all entries in the given day.
- Comment action – handles posting comments to a given entry.

In our poll application, we’ll have four views:

- Question “index” page – displays the latest few questions.
- Question “detail” page – displays a question text, with no results but with a form to vote.
- Question “results” page – displays results for a particular question.
- Vote action – handles voting for a particular choice in a particular question.

In Django, web pages and other content are delivered by views. 

Each view is represented by a simple Python function (or method, in the case of class-based views). 

Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name).

Now in your time on the web you may have come across such beauties as `“ME2/Sites/dirmod.asp?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B”`. 

You will be pleased to know that Django allows us much more elegant URL patterns than that.

### Django 支援多元素的Url

A URL pattern is simply the general form of a URL - for example: 

    /newsarchive/<year>/<month>/.

To get from a URL to a view, Django uses what are known as ‘URLconfs’. A URLconf maps URL patterns (described as regular expressions) to views.

This tutorial provides basic instruction in the use of URLconfs, and you can refer to [django.core.urlresolvers](https://docs.djangoproject.com/en/1.8/ref/urlresolvers/#module-django.core.urlresolvers) for more information.

## Write your first view

>>>> Goal:Create First View

polls/views.py

    from django.http import HttpResponse


    def index(request):
        return HttpResponse("Hello, world. You're at the polls index.")

Explanation :

>> To call the view, `we need to map it to a URL` - and for this we need a `URLconf`.

>> To create a `URLconf` in the polls directory, create a file called `urls.py`.

Your directory look like:

    polls/
        __init__.py
        admin.py
        models.py
        tests.py
        urls.py
        views.py

>> Create a file that call urls.py in polls app.

polls/urls.py

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        url(r'^$', views.index, name='index'),
    ]

Next step
    
    1.Point the root URLconf at the `polls.urls` module.

    2.Include URLconf in mysite/urls.py

mysite/urls.py

    from django.conf.urls import include, url
    from django.contrib import admin

    urlpatterns = [
        url(r'^polls/', include('polls.urls')),
        url(r'^admin/', include(admin.site.urls)),
    ]

Try to run

    http://localhost:8000/polls/

You will get the message 

    Hello, world. You’re at the polls index.

## url() function is passed four arguments

two required: `regex` and `view`, 
and two optional: `kwargs`, and `name`.

### url() argument: regex

The term “regex” is a commonly used short form meaning “regular expression”, which is a syntax for matching patterns in strings, or in this case, url patterns. Django starts at the first regular expression and makes its way down the list, comparing the requested URL against each regular expression until it finds one that matches.

Note that these regular expressions do not search GET and POST parameters, or the domain name. For example, in a request to http://www.example.com/myapp/, the URLconf will look for myapp/. In a request to http://www.example.com/myapp/?page=3, the URLconf will also look for myapp/.

If you need help with regular expressions, see Wikipedia’s entry and the documentation of the re module. Also, the O’Reilly book “Mastering Regular Expressions” by Jeffrey Friedl is fantastic. In practice, however, you don’t need to be an expert on regular expressions, as you really only need to know how to capture simple patterns. In fact, complex regexes can have poor lookup performance, so you probably shouldn’t rely on the full power of regexes.

Finally, a performance note: these regular expressions are compiled the first time the URLconf module is loaded. They’re super fast (as long as the lookups aren’t too complex as noted above).

### url() argument: view

When Django finds a regular expression match, Django calls the specified view function, with an HttpRequest object as the first argument and any “captured” values from the regular expression as other arguments. If the regex uses simple captures, values are passed as positional arguments; if it uses named captures, values are passed as keyword arguments. We’ll give an example of this in a bit.

### url() argument: kwargs

Arbitrary keyword arguments can be passed in a dictionary to the target view. We aren’t going to use this feature of Django in the tutorial.

### url() argument: name

Naming your URL lets you refer to it unambiguously from elsewhere in Django especially templates. This powerful feature allows you to make global changes to the url patterns of your project while only touching a single file.

## Writing more views

Add more view in views.py

    def detail(request, question_id):
        return HttpResponse("You're looking at question %s." % question_id)

    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)

    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)

New View into polls/urls.py

>> Add More view to polls/view.py

polls/views.py

    def detail(request, question_id):
        return HttpResponse("You're looking at question %s." % question_id)

    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)

    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)

New views into the polls.urls module by adding the following url() calls:

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        # ex: /polls/
        url(r'^$', views.index, name='index'),
        # ex: /polls/5/
        url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
        # ex: /polls/5/results/
        url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
        # ex: /polls/5/vote/
        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
    ]

try it

    /polls/34/          # it will run detail()

    /polls/34/results/  # it will run result()

    /polls/34/vote/     # it will run vote()

result in a call to the `detail` view like so :

    detail(request=<HttpRequest object>, question_id='34')

some thing like that

    url(r'^polls/latest\.html$', views.index),

don't do that.

## Write views that actually do something

>> Each view is responsible for doing one of two things: 

- returning an `HttpResponse object` containing the content for the requested page, 
- or `raising an exception` such as `Http404`.

Here’s one stab at a new index() view, which displays the latest 5 poll questions in the system, separated by commas, according to publication date:

polls/views.py

    from django.http import HttpResponse

    from .models import Question


    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        output = ', '.join([p.question_text for p in latest_question_list])
        return HttpResponse(output)

create a file in the polls/tempaltes/polls/index.html

polls/templates/polls/index.html

    {% if latest_question_list %}
        <ul>
        {% for question in latest_question_list %}
            <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No polls are available.</p>
    {% endif %}

let’s update our index view in `polls/views.py` to use the template:

polls/views.py

    from django.http import HttpResponse
    from django.template import RequestContext, loader

    from .models import Question


    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = RequestContext(request, {
            'latest_question_list': latest_question_list,
        })
        return HttpResponse(template.render(context))

A shortcut : `render()`

    from django.shortcuts import render

    from .models import Question


    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        context = {'latest_question_list': latest_question_list}
        return render(request, 'polls/index.html', context)

`polls/index.html` is mean the file in `templates/polls/index.html`

## Raising a 404 error

polls/views.py

    from django.http import Http404
    from django.shortcuts import render

    from .models import Question
    # ...
    def detail(request, question_id):
        try:
            question = Question.objects.get(pk=question_id)
        except Question.DoesNotExist:
            raise Http404("Question does not exist")
        return render(request, 'polls/detail.html', {'question': question})

The new concept here: The view raises the Http404 exception if a question with the requested ID doesn’t exist.

polls/templates/polls/detail.html

    {{ question }}

A shortcut : get_object_or_404()

polls/views.py

    from django.shortcuts import get_object_or_404, render

    from .models import Question
    # ...
    def detail(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})

## Use the template system

polls/detail.html

    <h1>{{ question.question_text }}</h1>
    <ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }}</li>
    {% endfor %}
    </ul>

## Removing hardcoded URLs in templates

polls/index.html

    <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>

this code hardcore the url.
using `{% url %}`
    
    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

It's 

    <a href="/polls/{{ question.id }}/">

Replace to be

    <a href="{% url 'detail' question.id %}">

polls.urls.py

    # the 'name' value as called by the {% url %} template tag
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

replace to be

    # added the word 'specifics'
    url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

## Namespacing URL names

mysite/urls.py

    from django.conf.urls import include, url
    from django.contrib import admin

    urlpatterns = [
        url(r'^polls/', include('polls.urls', namespace="polls")),
        url(r'^admin/', include(admin.site.urls)),
    ]

polls/index.html 

    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

replace be

    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>