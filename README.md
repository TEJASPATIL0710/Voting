# Voting
Project for YBIT


Certainly! Here's a step-by-step guide to creating a basic voting system using Python and Django, starting from installation.
1. Install Python and Django
First, ensure that Python is installed on your system. You can check by running:
bash
Copy code

python --version

If Python is not installed, download and install it from python.org.
Next, install Django using pip (Python's package installer):
bash
Copy code

pip install django

2. Create a Django Project
Once Django is installed, you can create a new project. Open your terminal or command prompt, navigate to the directory where you want to create your project, and run:
bash
Copy code

django-admin startproject voting_system

This command creates a new Django project named voting_system.
3. Create a Django App
Navigate into your project directory:
bash
Copy code

cd voting_system

Now, create a new app within your project. An app is a component of your project that handles specific functionality (like voting, in this case).
bash
Copy code

python manage.py startapp voting

This creates an app named voting.
4. Configure the Django Project
Open the project in your text editor or IDE. Modify the settings.py file in the voting_system directory to include your app in the INSTALLED_APPS section:
python
Copy code

# voting_system/settings.py

INSTALLED_APPS = [
    ...
    'voting',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

5. Create Models
In the voting app, define the models for Users, Candidates, and Votes in the models.py file:
python
Copy code

# voting/models.py

from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    is_admin = models.BooleanField(default=False)

class Candidate(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()

    def __str__(self):
        return self.name

class Vote(models.Model):
    voter = models.ForeignKey(User, on_delete=models.CASCADE)
    candidate = models.ForeignKey(Candidate, on_delete=models.CASCADE)
    timestamp = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('voter', 'candidate')

    def __str__(self):
        return f"{self.voter.username} voted for {self.candidate.name}"

6. Create and Apply Migrations
Django uses migrations to apply changes to the database schema. Create and apply these migrations:
bash
Copy code

python manage.py makemigrations
python manage.py migrate

7. Create an Admin Interface
To manage your models through Django's built-in admin interface, register your models in the admin.py file:
python
Copy code

# voting/admin.py

from django.contrib import admin
from .models import User, Candidate, Vote

admin.site.register(User)
admin.site.register(Candidate)
admin.site.register(Vote)

8. Create Forms
Create forms for user registration and voting in forms.py:
python
Copy code

# voting/forms.py

from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import User, Vote, Candidate

class UserRegistrationForm(UserCreationForm):
    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

class VoteForm(forms.ModelForm):
    class Meta:
        model = Vote
        fields = ['candidate']

9. Create Views
Create views to handle user registration, voting, and results in views.py:
python
Copy code

# voting/views.py

from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate
from django.contrib.auth.decorators import login_required
from .forms import UserRegistrationForm, VoteForm
from .models import Candidate, Vote

def register(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('vote')
    else:
        form = UserRegistrationForm()
    return render(request, 'register.html', {'form': form})

@login_required
def vote(request):
    if request.method == 'POST':
        form = VoteForm(request.POST)
        if form.is_valid():
            vote = form.save(commit=False)
            vote.voter = request.user
            vote.save()
            return redirect('results')
    else:
        form = VoteForm()
    candidates = Candidate.objects.all()
    return render(request, 'vote.html', {'form': form, 'candidates': candidates})

@login_required
def results(request):
    results = Vote.objects.values('candidate__name').annotate(count=models.Count('candidate')).order_by('-count')
    return render(request, 'results.html', {'results': results})

10. Create URL Routing
Set up the URL routes for the views in urls.py. First, create a urls.py file in your voting app if it doesn't already exist:
python
Copy code

# voting/urls.py

from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [
    path('register/', views.register, name='register'),
    path('login/', auth_views.LoginView.as_view(template_name='login.html'), name='login'),
    path('vote/', views.vote, name='vote'),
    path('results/', views.results, name='results'),
]

Then, include this urls.py in the project's main urls.py file:
python
Copy code

# voting_system/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('voting.urls')),
]

11. Create Templates
Create templates for registration, voting, and results in a templates directory inside the voting app.
register.html
html
Copy code

{% extends 'base.html' %}
{% block content %}
<h2>Register</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Register</button>
</form>
{% endblock %}
vote.html
html
Copy code
{% extends 'base.html' %}
{% block content %}
<h2>Vote for a Candidate</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Vote</button>
</form>
<ul>
    {% for candidate in candidates %}
    <li>{{ candidate.name }}</li>
    {% endfor %}
</ul>
{% endblock %}
results.html
html
Copy code
{% extends 'base.html' %}
{% block content %}
<h2>Voting Results</h2>
<ul>
    {% for result in results %}
    <li>{{ result.candidate__name }}: {{ result.count }} votes</li>
    {% endfor %}
</ul>
{% endblock %}

12. Create a Superuser
To manage the admin interface, create a superuser account:
bash
Copy code

python manage.py createsuperuser

Follow the prompts to set up the username, email, and password.
13. Run the Development Server
Finally, start the Django development server:
bash
Copy code

python manage.py runserver

14. Test Your Application
Access the Django admin interface at http://127.0.0.1:8000/admin/ to add candidates and manage data.
Register a user at http://127.0.0.1:8000/register/.
Log in at http://127.0.0.1:8000/login/.
Vote at http://127.0.0.1:8000/vote/.
View results at http://127.0.0.1:8000/results/.

15. Further Enhancements
Styling: Use Django's static files management to add CSS and JavaScript for better UI/UX.
Security: Implement additional security measures like password validation, CSRF protection, and user roles.
Features: Add more features like vote editing, vote tracking, and more detailed results analysis.
This guide gives you a working prototype of a basic voting system using Django. You can expand and customize it according to your needs.
