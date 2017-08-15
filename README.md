# Login-using-Django
Create a user using Django and Login to the web application
django-admin startproject login_app
django-admin startapp first_app

In settings.py

TEMPLATES_DIR=os.path.join(BASE_DIR,'templates')
MEDIA_DIR=os.path.join(BASE_DIR,'media')
STATIC_DIR=os.path.join(BASE_DIR,'static')

Add the app in INSTALLED_APPS list
In TEMPLATES list DIRS key, add the TEMPLATES_DIR folder

STATICFILES_DIRS=[STATIC_DIR,]

MEDIA_URL='/media/'
MEDIA_ROOT=MEDIA_DIR

To add powerful password algorithms argon2 and SHA256 **Needs to be installed before
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
]

Create these folders under base root folder
Create urls.py in first_app folder also

in root/urls.py file add
from django.conf.urls import url,include
from first_app import views

in URL Patterns
url(r'^index/',views.index,name='index'),
    url(r'^first_app/',include('first_app.urls'))
	
So whenever the first_app is included in the html path then it is routed to first_app/urls.py
in this urls add the below,
app_name='first_app' # this is to use template tagging from the html page like href="{% url "first_app:register" %}"

in views.py define the views that are included in the urls.py files

Next is to create the Register Form and the corresponding models to save the data
We already have a User model with the following chars
username,password,email,first_name,surname so that can be used in user creation

from django.contrib.auth.models import User

class Userprofile(models.Model):
	user=models.OneToOneField(User)
	website=models.URLField(blank=True) # to set the website not as mandatory
	profile_pic=models.ImageField(blank=True) # to use image field pip install pillow needs to be done
	
create forms.py in the first_app folder
from django import forms

class UserForm(forms.ModelForm):
	password=forms.charField(widget=forms.PasswordInput)
	class Meta():
		model=user
		fields=('username','password','email')
		
class UserProfileForm(forms.ModelForm):
	class Meta():
		model=UserProfile
		fields=('profile_pic','website')
		
Now we have created the models and the corresponding forms, next we have to show these forms in FE
In views.py

def register(request):
    registered=False
    if request.method=='POST':
        userform=UserForm(data=request.POST)
        userprofileform=UserProfileForm(data=request.POST)
        if userform.is_valid() and userprofileform.is_valid():
            user=userform.save()
            user.set_password(user.password)
            user.save()
            userprofile=userprofileform.save(commit=False)
            userprofile.user=user
            print('Before Profile Pic check')
            if 'profile_pic' in request.FILES:
                print('The File is present')
                userprofile.profile_pic=request.FILES['profile_pic']
            userprofile.save()
            registered=True
        else:
            print(userform.errors,userprofileform.errors)
    else:
         userform=UserForm()
         userprofileform=UserProfileForm()
    data_dict={'user_view':userform,'user_profile_view':userprofileform,'registered':registered}
    return render(request,'first_app/register.html',context=data_dict)
	
	To upload the images, we should have the form enctype as 
	<form method="post" enctype="multipart/form-data">
    {% if registered %}
    <h1>Thanks for Registering</h1>
    {%else%}
    <h1>Register:</h1>
    {%csrf_token%}
    {{user_view.as_p}}
    {{user_profile_view.as_p}}
    <input class="btn btn-primary" type="submit" name="register" value="Register">
    {%endif%}
  </form>
  
  To LOGIN and LOGOUT
  in urls.py create two url patterns for logging in and loggin out
  url(r'^login/',views.user_login,name='login'),
    url(r'^login/',views.user_logout,name='logout'),
	
in Views.py import the following
from django.contrib.auth import authenticate,login,logout
from django.http import HttpResponse,HttpResponseRedirect
from django.core.urlresolvers import reverse
from django.contrib.auth.decorators import login_required

@login_required
def user_logout(request):
    logout(request)
    return HttpResponseRedirect(reverse('index'))

def user_login(request):
    if request.method=='POST':
        user_name=request.POST.get('username')
        password=request.POST.get('password')
        user=authenticate(username=user_name,password=password)
        if user:
            if user.is_active:
                login(request,user)
                return HttpResponseRedirect(reverse('index'))
            else:
                return HttpResponse('Your acc is not active')
        else:
            print("Someone tried to login and failed.")
            print("They used username: {} and password: {}".format(username,password))
            return HttpResponse("Invalid login details supplied.")
    else:
        return render(request,'first_app/login.html',{})
		
A normal form for user login needs to be created in login.html and these names are acccesed above using the POST.get

In base.html user.is_authenticated need to be used to check if a user is logged in or not
  {% if user.is_authenticated %}
          <li><a href="{% url 'logout' %}">Logout</a></li>
          <li><a href="#">{{user.username}}</a></li>
        {% else %}
          <li><a class="navbar-link" href="{% url 'login' %}">Login</a></li>
        {% endif %}
