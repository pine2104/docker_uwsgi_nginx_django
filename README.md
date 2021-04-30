# Django-uWSGI-NGINX-Docker Application
[**Django**][1]-based web application and deploy with [**uWSGI**][2], [**NGINX**][3] and [**Docker**][4]

> **Django**
* **Add super-user**

    `$ python3 manage.py createsuperuser`
    
        Username: your_user_name
        Email address: email@gmail.com
        Password: *******
        Password (again): *******

* **Add environment variable**

    Use environment variable to store and get sensitive information.
    Do not recommend that directly type in setting.py rather than use `os.getenv('email_server')`.
    
    **email_server:**
  
    `$ export email_server="testemail@gmail.com"`
  
    `$ export email_password="password"`
    
    **SECRET_KEY:**
  You can generate your SECRET_KEY using [this][5].
  
    `$ export SECRET_KEY="!&(u2hn8bfs#v2ow_1!6=)olmud18%%ea@70gjld*7+@k=n)if"`


* **Check environment variable**

    `$ echo "$email_server"`
  
	`$ echo "$email_password"`

    `$ echo "SECRET_KEY"`



* **Run server**

  We can test it using default web-server in django. 
  
  `$ python manage.py runserver 0.0.0.0:port`


Because of the [bad performance][6], we will use [uWSGI][7] to deploy our web-application latter.

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

First, pick a plasmid to use as PCR template. 
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

Choose a pair of pirmers.

![Image][59]

And press `cal pcr` button.

![Image][60]

Result rendered.

![Image][61]

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

NGINX (engine X) is a web server that use for serving static files, e.g. CSS, images, .js files. 



* **Locate uwsgi_params at /etc/nginx**

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



* **Docker-compose file .yml**




> **Frequently used Linux command**

* **Check port**

    `$ sudo lsof -i:port`
* **Kill port**

    `$ sudo kill -9 $(sudo lsof -t -i:port)`




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
