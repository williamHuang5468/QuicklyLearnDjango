#Writing your first Django app

Goal 

	we’ll walk you through the creation of a basic poll application.

it has two part.

- A public site that lets people view polls and vote in them.
- An admin site that lets you add, change and delete polls.

## 1. step - Create probject

command :

django-admin startproject `projectName`

ex.

>> django-admin startproject mysite

## 2. Step - Database setup

	python manage.py migrate

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

Update

	python manage.py makemigrations polls

You will get message:

	Migrations for 'polls':
  		0001_initial.py:
    		- Create model Question
    		- Create model Choice
    		- Add field question to choice

run migrate again to create those model tables in your database:

	python manage.py migrate