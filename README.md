# Django_ChatGPT

we are going to create a live chat app with Chat GPT using Django and JavaScript.

Requirements:

    Basic understanding of Django
    Basic understanding of JavaScript and AJAX
    Basic understanding of 
    
## STEP 1:

    First we have to create a Django project:

        django-admin startproject Django_ChatGPT

    After that we have to create a application as well:

        cd Django_ChatGPT
        py manage.py startapp chat

## STEP 2:

    Now we have to setup our Django project’s settings.py and urls.py

    Head into the ChatGPT/settings.py file.

    Inside settings.py file you should see a INSTALLED_APPS. Add your application (chat) inside the list:

        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'chat', # Here you should add your app.
        ]