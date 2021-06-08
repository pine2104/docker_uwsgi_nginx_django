# Django-uWSGI-NGINX-Docker Application
[**Django**][1]-based web application and deploy with [**uWSGI**][2], [**NGINX**][3] and [**Docker**][4]

> **Prepare environment**

* **Create virtual environment**

  1.Install

  If you have not installed virtual environment,

  `$ sudo apt-get install python3-venv`

  2.Create

  `$ python3 -m venv /path/to/virtual/environment`
  
  ex:
  `$ python3 -m venv myvenv`

  3.Enter
  
  `$ source myvenv/bin/activate`

  If succeeded,

  `(myvenv) computer@user:~$ `


* **Install required packages in venv**

  `$ pip install -r path/to/requirements.txt`

> **Django**

  Templates: CSS, Bootstrap, BootstrapVue

  Functions: Register, Login, Logout, Forget password, Create/Delete/Update posts and Search.


* **Add super-user**

    `$ python3 manage.py createsuperuser`
    
        Username: your_user_name
        Email address: email@gmail.com
        Password: *******
        Password (again): *******

* **Add environment variable**

    Use environment variable to store and get sensitive information.
    Do not recommend that directly type in setting.py rather than use `os.getenv('email_server')`.
    
    1.**email_server:**
  
    `$ export email_server="testemail@gmail.com"`
  
    `$ export email_password="password"`
    
    2.**SECRET_KEY:**
  
    You can generate your SECRET_KEY using [this][5].
  
    `$ export SECRET_KEY="!&(u2hn8bfs#v2ow_1!6=)olmud18%%ea@70gjld*7+@k=n)if"`


* **Check environment variable**

    `$ echo "$email_server"`
  
	`$ echo "$email_password"`

    `$ echo "SECRET_KEY"`


* **Create Django project**

  `$ django-admin startproject sample`


* **Run server**

  We can test it using default web-server in django. 
  
  `$ python manage.py runserver 0.0.0.0:port`

  Because of the [bad performance][6], we will use [uWSGI][7] to deploy our web-application later on.


* **Create new app in your project**

  `$ python manage.py startapp primer`

  Now, you have created a primer-app 


* **Migrate Django model to the database**

  After finish designing models.py, you have to collect all changes,
  
  `$ python manage.py makemigrations`

  therefore, migrate to your database.

  `$ python manage.py migrate`


> **HTML and CSS**

* **Load static files**

  Under debug mode (`DEBUG = True` in settings.py), django will 
  automatically find your static files in sample/primer/static.
  
  You should add `{% load static %}` in the beginning of your .html,
  so load css file by 
  
  `<link rel="stylesheet" type="text/css" href="{% static 'posts/post.css' %}">
  `
  
  **1.Load .js file**
  
  `<script src="{% static 'main.js' %}" async></script>`

  **2.Load images**

  `<img src="{% static 'image.png' %}">`

> **My applications**

Some demonstrations show below.

* **Basic**

  **Register, Login, Logout, Forget password, Create/Delete/Update posts and Search posts.** 
  

* **Export primers**

  Enter admin page, and choose Primers model. 
  Export button will give you all primers fields in your database. 


![Image][50]

  There are several formats you can choose, ex: .csv, .xls, xlsx, ods,... 

![Image][51]

  Each column represents the fields in your django model. 

![Image][52]

* **Import primers**

  We can import thousands of primers at a time.
  The fields in sheet file should be the same with the exported table.

![Image][53]

* **Find primer pairs using primer and plasmid database**

  We can use primers and plasmids in our database to do PCR(polymerase chain reaction) analysis.

![Image][54]

  First, pick a plasmid as a PCR template. 
  If not exist, you can press `Create New Vector` button. 

![Image][55]

  `Set Length of PCR` means your length of desired PCR product.

  `Tolerated Length of PCR` means your length of error range.

  If `Set Length of PCR` = 500, `Tolerated Length of PCR` = 100, results will give you Length between 500-100 to 500+100.


  Pick one primer and press `cal pcr` button.

![Image][56]

  Results rendered.
![Image][57]

* **Find sequence of PCR product of primer pairs**

  If you want to check your PCR detail, you can use `Select Pairs` button.

![Image][58]

  Choose a pair of primers.

![Image][59]

  Next, press `cal pcr` button.

![Image][60]

  Result rendered.

![Image][61]


* **Upload files with progress bar**
  

  1.Upload multiple files at a time

  models.py,
  ```
  from django.db import models
  
  # Create your models here.
  
  class Upload(models.Model):
      upload_file = models.FileField() # .name, .size, .url, .open, .close, .save, .delete,
      upload_date = models.DateTimeField(auto_now_add =True)
  ```
  forms.py,
  ```
  from django import forms
  from django.forms import ClearableFileInput
  from .models import Upload
  
  
  class FileUpload(forms.ModelForm):
      class Meta:
          model = Upload
          fields = ['upload_file']
          widgets = {
              'upload_file': ClearableFileInput(attrs={'multiple': True}),
          }
  ```

  views.py,

  ```
  from django.shortcuts import render
  from django.contrib.auth.decorators import login_required
  from .models import Upload
  from .forms import FileUpload
  
  @login_required #only when user log in, this function works
  def upload_file(request):
      all_files = Upload.objects.all() # for listing all stored files
      if request.method == 'POST':
           form = FileUpload(request.POST, request.FILES)
           files = request.FILES.getlist('upload_file')
           if form.is_valid():
               for f in files: # save each file sequentially
                   file_instance = Upload(upload_file=f)
                   file_instance.save()
      else:
           form = FileUpload()
  
      return render(request, 'uploader/upload_files.html', {'form': form, 'all_files':all_files})
  ```

  upload_files.html,
  
  ```
  {% extends "posts/base.html" %}
  {% load crispy_forms_tags %}
  {% block content %}
  
      <div id="alert-box"></div>
      <div id="image-box"></div>
  
      <h1 class="jc-title" style="font-size: 5.0vmin;">Upload Files </h1>
      <h1 class="article-header">
          Use for directly transmitting data or files. Please delete it after finishing work for saving storage .
      </h1>
      <form id="upload-form" method="post" enctype="multipart/form-data">
          {% csrf_token %}
          {{ form.upload_file }}
          <button id="upload" type="submit" class="btn btn-primary">Upload Files</button>
      </form>
  
      <div id="progress-box" class="not-visible">progress</div>
      <div id="cancel-box" class="not-visible">
          <button id="cancel-btn" class="btn btn-danger">cancel</button>
      </div>
  
      <div class="containter">
      <ul>
          {% for document in all_files %}
              <li>
  
                  <a href="{{ document.upload_file.url }}">{{ document.upload_file.name }}</a>
                  <small>({{ document.upload_file.size|filesizeformat }}) - {{document.upload_date}}</small>
  
                  <a href="{% url 'delete' document.pk %}">delete</a>
  
              </li>
          {% endfor %}
          </ul>
  
      </div>
  {% endblock content %}
  ```

  2.Progress bar
  Use javascript to listen the progress of data transmitting.
  
![Image][62]

  in /sample/uploader/static/main.js,

  ```
  const uploadForm = document.getElementById('upload-form')
  const input = document.getElementById('upload')
  const file = document.getElementById('id_upload_file')
  
  console.log(input)
  
  const alertBox = document.getElementById('alert-box')
  const imageBox = document.getElementById('image-box')
  const progressBox = document.getElementById('progress-box')
  const cancelBox = document.getElementById('cancel-box')
  const cancelBtn = document.getElementById('cancel-btn')
  
  const csrf = document.getElementsByName('csrfmiddlewaretoken')
  
  input.addEventListener('click', ()=>{
      progressBox.classList.remove('not-visible')
      cancelBox.classList.remove('not-visible')
  
      const img_data = file.files[0]
      const url = URL.createObjectURL(img_data)
      console.log(img_data)
      const fd = new FormData()
      fd.append('csrfmiddlewaretoken', csrf[0].value)
      fd.append('image', img_data)
      $.ajax({
          type:'POST',
          url: uploadForm.action,
          enctype: 'multipart/form-data',
          data: fd,
          beforeSend: function(){
              console.log('before')
              alertBox.innerHTML= ""
              imageBox.innerHTML = ""
          },
          xhr: function(){
              const xhr = new window.XMLHttpRequest();
              xhr.upload.addEventListener('progress', e=>{
                  // console.log(e)
                  if (e.lengthComputable) {
                      const percent = e.loaded / e.total * 100
                      console.log(percent)
                      progressBox.innerHTML = `<div class="progress">
                                                  <div class="progress-bar" role="progressbar" style="width: ${percent}%" aria-valuenow="${percent}" aria-valuemin="0" aria-valuemax="100"></div>
                                              </div>
                                              <p>${percent.toFixed(1)}%</p>`
                  }
  
              })
              cancelBtn.addEventListener('click', ()=>{
                  xhr.abort()
                  setTimeout(()=>{
                      uploadForm.reset()
                      progressBox.innerHTML=""
                      alertBox.innerHTML = ""
                      cancelBox.classList.add('not-visible')
                  }, 2000)
              })
              return xhr
          },
          success: function(response){
              console.log(response)
              imageBox.innerHTML = `<img src="${url}" width="300px">`
              alertBox.innerHTML = `<div class="alert alert-success" role="alert">
                                      Successfully uploaded below
                                  </div>`
              cancelBox.classList.add('not-visible')
          },
          error: function(error){
              console.log(error)
              alertBox.innerHTML = `<div class="alert alert-danger" role="alert">
                                      Ups... something went wrong
                                  </div>`
          },
          cache: false,
          contentType: false,
          processData: false,
      })
  }) 
  ```



> **uWSGI**

  Now, we can use uWSGI to deploy our application to get better performance.
  
  uWSGI is not good at serving static files, so NGINX is required for serving static files.

* **Setting for uwsgi.ini file**

  Ues [unix domain sockets][8] instead of TCP ports to communicate with NGINX. 
  Unix domain sockets is faster and lighter than TPC/IP socket.

  ```
  [uwsgi]
  socket= app.sock
  master=true
  processes=4
  threads=2
  module=sample.wsgi:application
  vacuum = true
  ```


* **Launch uWSGI using uwsgi.ini**

  Enter command below to initiate uWSGI with uwsgi.ini setting.

  `$ uwsgi --ini uwsgi.ini`


> **NGINX**

NGINX is a web server with high efficiency for serving static files, e.g. CSS, images, .js files. 

* **Install**

  `$ sudo apt-get install nginx`


* **Uninstall**

  Completely remove followed by three steps.

  1.`$ sudo apt-get remove nginx nginx-common`
  
  2.`$ sudo apt-get purge nginx nginx-common`
  
  3.`$ sudo apt-get autoremove`


There are some required settings for convenience of using NGINX.

* **Locate uwsgi_params at /etc/nginx**

  Required file for uWSGI.
  ```
  uwsgi_param QUERY_STRING $query_string;
  uwsgi_param REQUEST_METHOD $request_method;
  uwsgi_param CONTENT_TYPE $content_type;
  uwsgi_param CONTENT_LENGTH $content_length;
  uwsgi_param REQUEST_URI $request_uri;
  uwsgi_param PATH_INFO $document_uri;
  uwsgi_param DOCUMENT_ROOT $document_root;
  uwsgi_param SERVER_PROTOCOL $server_protocol;
  uwsgi_param REMOTE_ADDR $remote_addr;
  uwsgi_param REMOTE_PORT $remote_port;
  uwsgi_param SERVER_ADDR $server_addr;
  uwsgi_param SERVER_PORT $server_port;
  uwsgi_param SERVER_NAME $server_name;
  ```


* **Replace nginx.conf at /etc/nginx**

    Directly change nginx.conf to ignore default.conf
    and NGINX will read sample.conf in /etc/nginx/sites-enabled/
  ```	
  # comment this line
  # include /etc/nginx/conf.d/*.conf;
  
  include /etc/nginx/sites-enabled/*;
  ```

* **Link sample.conf to /etc/nginx/sites-enabled**

    `ln -s /path/to/sample.conf /etc/nginx/sites-enabled/sample.conf`


> **Docker and Docker-compose**


* **Dockerfile**

  During deploying stage, 
  you can hide your sensitive information with `os.getenv('variable')`.
  
  Therefore, in Django setting.py, 
  ```
  SECRET_KEY = os.getenv('SECRET_KEY')
  EMAIL_HOST_USER = os.getenv('email_server')
  EMAIL_HOST_PASSWORD = os.getenv('email_password')
  ```

1. django_application (sample/Dockerfile)
    ```
    FROM python:3.8.9
    LABEL maintainer="ychanc2104@gmail.com"
    ENV PYTHONUNBUFFERED=1
    ENV email_server="testemail@gmail.com"
    ENV email_password="password"
    ENV SECRET_KEY="!&(u2hn8bfs#v2ow_1!6=)olmud18%%ea@70gjld*7+@k=n)if"
    RUN mkdir /docker_api
    WORKDIR /docker_api
    COPY . /docker_api
    RUN pip install --no-cache-dir -r requirements.txt
    ```
    Change ENV to meet your email_server and secret key.
   

2. server (proxy/Dockerfile)
  
    ```
    FROM nginx:latest
    LABEL maintainer="ychanc2104@gmail.com"
    
    COPY nginx.conf /etc/nginx/nginx.conf
    COPY sample.conf /etc/nginx/sites-available/sample.conf
    COPY uwsgi_params /etc/nginx/uwsgi_params
    
    RUN mkdir -p /etc/nginx/sites-enabled/ && \
        ln -s /etc/nginx/sites-available/sample.conf /etc/nginx/sites-enabled/sample.conf
    
    CMD ["nginx", "-g", "daemon off;"]
    ```

* **Docker-compose file .yml**
  
  To run multiple Dockerfiles.

  ```
  version: '3.8'
  services:
    proxy:
      container_name: dj3_nginx
      build: ./proxy
      restart: always
      ports:
        - "8000:8000"
      volumes:
        # Using the named volume from the Django project.
        - api_data:/docker_api
        - ./log:/var/log/nginx
      depends_on:
        - app
  
    app:
      container_name: dj3_web
      build: ./sample
      restart: always
      command: uwsgi --ini uwsgi.ini
      volumes:
        - api_data:/docker_api
      environment:
          - PYTHONUNBUFFERED=TURE
  
  volumes:
    api_data:
  ```

  Volume can be used to mount files in the container to your host-computer.
  Therefore, you can synchronize files in your host-computer.

> **Frequently used Linux command**

* **Check port**

    `$ sudo lsof -i:port`
* **Kill port**

    `$ sudo kill -9 $(sudo lsof -t -i:port)`

> **Frequently used Docker command**

* **List all containers**

    `$ docker ps -aq`

* **Stop all running containers**

    `$ docker stop $(docker ps -aq)`

* **Remove all containers**

    `$ docker rm $(docker ps -aq)`

* **Remove all images**

    `$ docker rmi $(docker images -q)`

* **Force Remove images**

    `$ docker images | grep ID | awk '{print $1 ":" $2}' | xargs docker rmi`

* **Remove all volumes**

    `$ docker volume rm $(docker volume ls)`



[1]: https://www.djangoproject.com/start/
[2]: https://uwsgi-docs.readthedocs.io/en/latest/
[3]: https://www.nginx.com/
[4]: https://docs.docker.com/
[5]: https://djecrety.ir/
[6]: https://docs.djangoproject.com/en/3.2/howto/deployment/
[7]: https://docs.djangoproject.com/en/3.2/howto/deployment/wsgi/
[8]: https://en.wikipedia.org/wiki/Unix_domain_socket

[50]: doc/img/export_primer_1.png
[51]: doc/img/export_primer_2.png
[52]: doc/img/import_primer_1.png
[53]: doc/img/import_primer_2.png
[54]: doc/img/seq_extr_1.png
[55]: doc/img/set_L_1.png
[56]: doc/img/set_L_2.png
[57]: doc/img/set_L_3.png
[58]: doc/img/set_pairs_1.png
[59]: doc/img/set_pairs_2.png
[60]: doc/img/set_pairs_3.png
[61]: doc/img/set_pairs_4.png
[62]: doc/img/progress_bar.png
