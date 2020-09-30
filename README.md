# [Deploy Flask App](https://dev.to/sm0ke/deploy-flask-app-complete-information-and-samples-4h6i)

This article (originally published on Dev.to) explains how to Deploy Flask in production using multiple configurations (Nginx/Gunicorn, Nginx/Waitress, Docker), and different sample apps. The concepts can be adapted with a minimum effort also on Django, the Flask's big brother.

<br />

## [What is Deployment](https://en.wikipedia.org/wiki/Software_deployment)

Software deployment is all of the activities that make a software system available for use. Most of the time, developers code their apps using a workstation and later publish the product on a public domain/subdomain to be used by end-users. The process of making the software available for *production* use is called deployment.

<br />

## [What is Flask](https://docs.appseed.us/what-is/flask/)

For newcomers, Flask is a lightweight **WSGI** web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. 
Classified as a microframework, **Flask** is written in Python and it does not require particular tools or libraries. It has no database abstraction layer, form validation, or any other components where pre-existing third-party libraries provide common functions.

**WSGI** (Web Server Gateway Interface) it is just an interface specification by which server and application communicate. 

Flask, for instance, implements this communication protocol under the hood and let us code the upper level of our web app (features, login, database information manipulation). 

## WSGI Simple App (no Flask)

Probably the most simple WSGI app ever coded is this one: 

```python
def app(environ, start_response):
    data = b"Hello! This is a simple WSGI web app.\n"
    start_response("200 OK", [
        ("Content-Type", "text/plain"),
        ("Content-Length", str(len(data)))
    ])
    return iter([data])
```

> Please save this code snippet in a file named `myapp.py` (or choose another name). We will refer this file in the next section. 

This code snippet defines a method named `app` (we can choose another name) that returns a simple string `Hello! This is a simple WSGI web app`

The syntax used to define the method and the response pattern is defined in the WSGI protocol, section **[The Application/Framework Side](https://www.python.org/dev/peps/pep-0333/#the-application-framework-side)**. 

On the other side, the server should also respect the WSGI protocol and implement **[The Server/Gateway Side](https://www.python.org/dev/peps/pep-0333/#the-server-gateway-side)**. 

This complex part is handled by Flask, Gunicorn, uWSGI, and all other WSGI-ready servers that we can use in production.

<br />

## [Gunicorn](https://gunicorn.org/)

At this point, we have a super simple WSGI app that echos a `Hello world!` .. and that's all. To see the message in the browser we need a WSGI server (Gunicorn) able to execute our app and serve browser request. 

**Install Gunicorn**

The most convenient way is to use PIP, the official package manager for Python. 

```bash
$ pip install gunicorn
```

Once the [Gunicorn](https://gunicorn.org/) is available we can execute our simple app.

```bash
$ gunicorn myapp:app
```

Gunicorn being a WSGI server, expect the WSGI object to execute as argument, witch in our case is defined in `myapp.py` file in method `app`. 

---

By default `Gunicorn` starts on port 8000. To use another port, use `-b` argument (`b` comes from `bind`).

```bash
$ gunicorn myapp:app -b :8001
``` 

**The Browser effect**

![Gunicorn - start App](https://raw.githubusercontent.com/admin-dashboards/deploy-flask-app/master/media/gunicorn-app-browser.jpg)

<br />

## [Waitress](https://docs.pylonsproject.org/projects/waitress/en/stable/)

Gunicorn is for sure an amazing server that we can use without issues on production .. but has a small limitation. Cannot be used on windows stations. This problem is solved by **Waitress**, a powerful WSGI-ready server. 

**Install waitress**

```bash
$ pip install waitress
```

We can execute our simple WSGI app under waitress using a single line:

```bash
$ waitress-serve myapp:app 
```

By default, **waitress** starts on port 8080, but we can adjust the port using `--port` argument:

```bash
$ waitress-serve --port=8000 myapp:app
```

That was pretty simple, right?

<br />

## WSGI App - Flask version

If we take a second look at `myapp.py`, we can see that our code delivers only a simple text and nothing else. How about setting a database, a fancy UI, some forms, security maybe? 

Well, using Flask we can re-use many modules and features coded by open-source enthusiasts across the globe. 

Before using `Flask`, we need to install it. `PIP` comes to the rescue:

```bash
$ pip install flask
```
 
A super simple (WSGI) Flask app might be this one: 

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello WSGI ... from Flask!'
```
 
> Save the content as `flask_app.py` for later reference. 


**flask_app.py** does the following:

- Import the Flask module
- Defines the WSGI object with the name `app`
- Define a default `route` that returns a simple string

---

**Execute with Gunicorn**

```bash
$ gunicorn flask_app:app
[3445] [INFO] Starting gunicorn 19.10.0
[3445] [INFO] Listening at: http://127.0.0.1:8000 (3445)
[3445] [INFO] Using worker: sync
[3449] [INFO] Booting worker with pid: 3449
```

**The Browser effect**

![Gunicorn - execute Flask app.](https://raw.githubusercontent.com/admin-dashboards/deploy-flask-app/master/media/gunicorn-flask-app-browser.jpg)

<br />

Execute using **waitress** is super simple. We need to write just a single line:

```bash
$ waitress-serve --port=8000 flask_app:app
```

If we visit the app in the browser, we should see the same `dummy` message `Hello WSGI ... from Flask!`.

<br />

## Nginx 

At this point, we are able to start the app using Gunicorn and Waitress but the production environment requires a secure HTTPS connection, the recommended way to expose services in current times. To achieve this, we need a `proxy` server configured in front of Gunicorn/Waitress to secure the connection with the browser. Nginx can be used as `reverse-proxy`. 

For newcomers, A [reverse proxy](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/) is a server that sits in front of web servers and forwards client (e.g. web browser) requests to those web servers. Reverse proxies are typically implemented to help increase security, performance, and reliability.  

Cloudfare provides a good intro to this useful concept - [What Is A Reverse Proxy](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/).

![Forward proxy flow - Reverse proxy architecture.](https://www.cloudflare.com/img/learning/cdn/glossary/reverse-proxy/forward-proxy-flow.svg)

<br />

**How to set up Nginx** 

Before using Nginx, we need to install it. If your production server uses Ubuntu, here is the magic line:

```bash
$ sudo apt install nginx
``` 

On CentOS servers, the syntax is slightly different:

```bash
$ sudo yum install epel-release
$ sudo yum install nginx
```

<br />

**Start the Nginx server**

```bash
$ systemctl status nginx
```

Nginx now serves a welcome page, and we need to add a simple configuration to link the default port `80` to forward the requests back & fort to our WSGI app. 

If we assume that WSGI app runs on port `8000` using Gunicorn or Waitress, Nginx must be configured to bind that address. We can do this with ease, by editing a single file.

```bash
$ vi /etc/nginx/sites-enabled/default  [On Debian/Ubuntu]
$ vi /etc/nginx/nginx.conf             [On CentOS/RHEL]
```

Before editing anything, is recommended to backup the current file, in case something is not configured properly. Once we did this, the file should contain the following snippet:

```config
server {
    listen      80;
    server_name www.my-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        include proxy_params;
    }
}
``` 

To load this new configuration, we need to restart Nginx: 

```bash
$ sudo systemctl reload nginx
```

By accessing the port `80` in the browser, we should see the same message previously served by the WSGI apps. 

<br />

With all those concepts understood, we can move further with this presentation and mention a few starters already configured for Nginx/Gunicorn, HEROKU, and Docker.
In case you have issues using the samples, just drop any questions in the comments area or join the AppSeed [Discord](https://discord.gg/fZC6hup) server and ask for help.

<br />

## [Flask Dashboard - Black Design](https://appseed.us/admin-dashboards/flask-dashboard-black)

Open-Source admin dashboard coded in **Flask Framework** - Features: 

- **[LIVE Demo](https://flask-dashboard-black.appseed.us/)**
- **[Product Page](https://appseed.us/admin-dashboards/flask-dashboard-black)**
- DBMS: SQLite, PostgreSQL (production) 
- DB Tools: SQLAlchemy ORM, Alembic (schema migrations)
- Modular design with **Blueprints**
- Session-Based authentication (via **flask_login**), Forms validation
- Deployment scripts: Docker, Gunicorn / Nginx, Heroku

<br />

![Flask Dashboard - Black Design, dashboard screen.](https://raw.githubusercontent.com/app-generator/flask-black-dashboard/master/media/flask-black-dashboard-screen.png)

<br />

## [Flask Dashboard Datta Able](https://appseed.us/admin-dashboards/flask-dashboard-dattaable)

Open-Source admin dashboard coded in **Flask Framework** - Features: 

- **[LIVE Demo](https://flask-dashboard-dattaable.appseed.us/)**
- **[Product Page](https://appseed.us/admin-dashboards/flask-dashboard-dattaable)**
- DBMS: SQLite, PostgreSQL (production) 
- DB Tools: SQLAlchemy ORM, Alembic (schema migrations)
- Modular design with **Blueprints**
- Session-Based authentication (via **flask_login**), Forms validation
- Deployment scripts: Docker, Gunicorn / Nginx, Heroku

<br />

![Flask Dashboard DattaAble - Starter project coded in Flask.](https://raw.githubusercontent.com/app-generator/flask-dashboard-dattaable/master/media/flask-dashboard-dattaable-screen.png)

<br />

## [Flask Corona Dark](https://appseed.us/admin-dashboards/flask-dashboard-corona-dark)

Open-Source admin dashboard coded in **Flask Framework** - Features: 

- **[LIVE Demo](https://flask-dashboard-corona-dark.appseed.us/)**
- **[Product Page](https://appseed.us/admin-dashboards/flask-dashboard-corona-dark)**
- DBMS: SQLite, PostgreSQL (production) 
- DB Tools: SQLAlchemy ORM, Alembic (schema migrations)
- Modular design with **Blueprints**
- Session-Based authentication (via **flask_login**), Forms validation
- Deployment scripts: Docker, Gunicorn / Nginx, Heroku

<br />

![Flask Dashboard Corona Dark - Open-Source template project provided by AppSeed.](https://raw.githubusercontent.com/app-generator/flask-dashboard-corona-dark/master/media/flask-dashboard-corona-dark-screen.png)

<br />

## [Flask Atlantis Dark](https://appseed.us/admin-dashboards/flask-dashboard-atlantis-dark)

Open-Source admin dashboard coded in **Flask Framework** - Features: 

- **[LIVE Demo](https://flask-dashboard-atlantis-dark.appseed.us/)**
- **[Product Page](https://appseed.us/admin-dashboards/flask-dashboard-atlantis-dark)**
- DBMS: SQLite, PostgreSQL (production) 
- DB Tools: SQLAlchemy ORM, Alembic (schema migrations)
- Modular design with **Blueprints**
- Session-Based authentication (via **flask_login**), Forms validation
- Deployment scripts: Docker, Gunicorn / Nginx, Heroku

<br />

![Flask Dashboard Atlantis Dark - Starter project coded in Flask.](https://raw.githubusercontent.com/app-generator/flask-dashboard-atlantis-dark/master/media/flask-dashboard-atlantis-dark-screen.png)

<br />

## [Flask Dashboard CoreUI](https://appseed.us/admin-dashboards/flask-dashboard-coreui)

**CoreUI** is an Open Source Bootstrap Admin Template. But CoreUI is not just another Admin Template. It goes way beyond hitherto admin templates thanks to transparent code and file structure. And if that's not enough, let’s just add that CoreUI consists bunch of unique features and over 1000 high-quality icons.

- **[LIVE Demo](https://flask-dashboard-coreui.appseed.us/)**
- **[Product Page](https://appseed.us/admin-dashboards/flask-dashboard-coreui)**
- DBMS: SQLite, PostgreSQL (production) 
- DB Tools: SQLAlchemy ORM, Alembic (schema migrations)
- Modular design with **Blueprints**
- Session-Based authentication (via **flask_login**), Forms validation
- Deployment scripts: Docker, Gunicorn / Nginx, Heroku

<br />

![Flask Dashboard CoreUI - Template project provided by AppSeed.](https://raw.githubusercontent.com/app-generator/flask-dashboard-coreui/master/media/flask-dashboard-coreui-screen.png)

<br />

### Resources

- [Flask](https://palletsprojects.com/p/flask/)  - the official website
- [What is Flask](https://docs.appseed.us/what-is/flask/) - a short introduction to this amazing framework
- More [Flask Apps](https://appseed.us/apps/flask-apps) and [Flask Dashboards](https://appseed.us/admin-dashboards/flask) provided by **AppSeed**.

<br />

---
[Deploy Flask App](https://dev.to/sm0ke/deploy-flask-app-complete-information-and-samples-4h6i) - Article provided by AppSeed
