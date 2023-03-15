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

Add static files in settings.py file:

    STATIC_URL = 'static/'
    STATICFILES_DIRS=[
        os.path.join(BASE_DIR, 'chat/static'),
    ]

Note: don’t forget to “import os” in settings.py file

Then head to ChatGPT/urls.py and include your applications urls as well:

    from django.contrib import admin
    from django.urls import path, include # Import 'include'

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include("chat.urls")), # Add this line
    ]

Note: don’t forget to add a urls.py file inside your chat (application) folder. Django does not create a urls.py by default.

Your chat/urls.py file should look like this (similar to ChatGPT/urls.py):

    from django.urls import path
    from .views import *


    urlpatterns = [
        path('', chat, name='chat'),
        path('ajax', Ajax, name='ajax'),
    ]

Head to your chat/models.py file and add this class:

    from django.db import models

    class Chat(models.Model):
        text = models.CharField(max_length=500)
        gpt = models.CharField(max_length=17000)
        date = models.DateTimeField(auto_now_add=True, blank=True, null=True)

Then go to your chat/views.py file and add these functions:

    from django.shortcuts import render
    from .models import *
    from django.http import JsonResponse
    from django.views.decorators.csrf import csrf_exempt
    import openai

    def chat(request):
        chats = Chat.objects.all()
        return render(request, 'chat.html', {
            'chats': chats,
        })

    @csrf_exempt
    def Ajax(request):
        if request.headers.get('X-Requested-With') == 'XMLHttpRequest': # Check if request is Ajax

            text = request.POST.get('text')
            print(text)

            openai.api_key = "YOUR_API_KEY" # Here you have to add your api key.
            res = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": f"{text}"}
            ]
            )

            response = res.choices[0].message["content"]
            print(response)

            chat = Chat.objects.create(
                text = text,
                gpt = response
            )

            return JsonResponse({'data': response,})
        return JsonResponse({})

Note: don’t forget to add your openai account API key

## STEP 3:

Inside your chat (application) folder create two folders:

static
templates
After that create a javascript.js file in static.

Also add layout.html and chat.html inside your templates file.

Your file structure should look like this:

    .
    └── Your_Computer_Folder/
        ├── Django_ChatGPT
        └── chat/
            ├── static/
            │   └── javascript.js
            └── templates/
                ├── layout.html
                └── chat.html

The javascript.js file should look like this:

    document.addEventListener('DOMContentLoaded', function(){

        document.querySelector('#submitBtn').addEventListener('click', () => chat_ajax());

    });

    function chat_ajax(){

        let text = document.querySelector('#userText').value
        let chatCard = document.querySelector('#chatCard')
        chatCard.innerHTML += `
        <div class="card-body bg bg-primary">
            <h5 class="card-title">${text}</h5>
        </div>
        `
        console.log(text)

        // Clear input:
        document.querySelector('#userText').value = null

        var loading = document.querySelector('#loading')
        loading.innerHTML = `
        <strong>Loading...</strong>
        <div class="spinner-border ms-auto" role="status" aria-hidden="true"></div>
        `

        $.ajax({
            type: 'POST',
            url: '/ajax',
            data: {
                'text': text
            },
            success: (res)=> {
                let response = res.data
                chatCard.innerHTML += `
                <div class="card-body bg bg-light text-dark">
                    <h5 class="card-title">${response}</h5>
                </div>
                `
                loading.innerHTML = ''
            },
            error: ()=> {
                console.log("There Was An Error!")
            }
        })
    }

Your layout.html should look like this:

    {% load static %}
    <!DOCTYPE html>
    <head lang="en">
        {% block head %}

        {% endblock %}
        <!-- CSS only -->
        <link href="{% static 'bootstrap/css/bootstrap.min.css' %}" rel="stylesheet">
        <!-- JavaScript Bundle with Popper -->
        <script type="text/javascript" src="{% static 'popper.min.js' %}" defer></script>
        <script type="text/javascript" src="{% static 'bootstrap/js/bootstrap.min.js' %}" defer></script>
        <!--AJAX-->
        <script type="text/javascript" src="{% static 'ajax.js' %}" defer></script>
        <!--My JS-->
        <script src="{% static 'javascript.js' %}" type="text/javascript" defer></script>
        <!--My CSS-->
        <link rel="stylesheet" href="{% static 'styles.css' %}">

    </head>
    <body class="bg bg-gradient bg-light">
    {% block body %}
    {% endblock %}
    </body>

Note: in the head tag you can import popper, ajax, and bootstrap using other methods but its better to download them and use local files for website speed (as implemented in Github).

Your chat.html should look like this:

    {% extends 'layout.html' %}
    {% load static %}

    {% block head %}
    <title>ChatGPT</title>
    {% endblock %}

    {% block body %}

    <section class="bg bg-gradient bg-light">
    <div class="container py-5">
    <div class="card">
    <div class="card-body">
        <div class="row d-flex justify-content-center">
    <div class="col-md-7 col-xl-5 mb-4 mb-md-0">
        <div class="py-4 d-flex flex-row">
        <h1>ChatGPT</h1>
        </div>
        <div class="card text-white" id="chatCard">
        {% for chat in chats %}
        <div class="card-body bg bg-primary">
        <h5 class="card-title">{{ chat.text }}</h5>
        </div>
        <div class="card-body bg bg-light text-dark">
        <h5 class="card-title">{{ chat.gpt }}</h5>
            </div>
        {% endfor %}
        </div>

        <!--Loading-->
        <div class="d-flex align-items-center mt-3 text-primary" id="loading">
        </div>
        <!--Loading End-->
        
        <div class="mt-3">
        <div class="form-field">
        <textarea id="userText" required class="form-control" type="textarea" name="address" placeholder="Text:"></textarea>
        </div>
        </div>
                    
        <div class="d-grid gap-2 mt-4">
        <button id="submitBtn" class="btn btn-primary fs-5">Send</button>
        </div>

        <br>
    
        </div>
    </div>  
        </div>
    </div>
    </div>
    </div>
    </section>

    {% endblock %}

Run:

    python manage.py makemigrations
    python manage.py migrate
    python manage.py runserver

<b>Congratulations! You have a Django web application capable of chatting with ChatGPT.</b>

<b>Summary</b>

<b>References</b>

https://getbootstrap.com/

https://openai.com/

https://jquery.com/download/

https://www.djangoproject.com/