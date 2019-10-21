---
layout: post
title: Deploy Jupyter on Heroku
date: 2019-09-23
excerpt: ""
tags: python, IDE
comments: true
---


# Deploy Jupyter on Heroku with Docker

> 參考:https://www.codingforentrepreneurs.com/blog/jupyter-production-server-on-docker-heroku/
> 

### 1. install pipenv
```python=
pipenv install jupyter tornado
pipenv run jupyter notebook
```

### 2. Add Production-read configuration for Jupyter / Docker deployment
Jupyter allows for all kinds of configuration. We're going to be doing minimal configuration so our notebook server is at least password protected.
```python=
mkdir conf
touch conf/notebook_config.py
```
In notebook_config.py, all you need to add:
```python=
import os

c = get_config()

# Kernel config
c.IPKernelApp.pylab = 'inline'  # if you want plotting support always in your notebook

# Notebook config

c.NotebookApp.allow_origin = '*' # put your public IP Address here
c.NotebookApp.ip = '*'
c.NotebookApp.allow_remote_access = True
c.NotebookApp.open_browser = False

# ipython -c "from notebook.auth import passwd; passwd()"
c.NotebookApp.password = u'sha1:67478c7bc2ab:d451b0be82a5ee051a6f13d84d301e961bfe1aec'
c.NotebookApp.port = int(os.environ.get("PORT", 8888))
c.NotebookApp.allow_root = True
c.NotebookApp.allow_password_change = True
```
> To create your own password, run ipython -c "from notebook.auth import passwd; passwd()", and set that result in the c.NotebookApp.password parameter above.
> 
> Above is a modified config file. To create the default jupyter notebook config file, run jupyter notebook --generate-config
> 

## 3. Create a Heroku App



## 4. Login to the Heroku Container Registry

```python=
heroku container:login
```

## 5. Push & Release
* push
```python=
heroku container:push web
```
* release
> Release If all went well with the push, you can release it:
```python=
heroku container:release web
```

>  Heroku filesystem of the dyno is ephemeral......
