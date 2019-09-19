 ---
layout: post
title: "Deploy Flask to Heroku"
date: 2019-09-19
excerpt: "flask with crawler to heroku"
tags: [chatbot, heroku, flask]
comments: true
---
# Deploy Flask to cloud(heroku)

- echo "python-3.5.3">runtime.txt
- 定義要安裝的封包清單
```python=
pip freeze > requirements.txt
```

- 查看佔用中的port process

```
lsof -i:[port]
kill 指定process
```

- 設定本機端的測試server



### 使用Pipenv

- 啟動環境
```python=
pipenv shell

#安裝 requirement
pip install -r requirements.txt

#安裝套件
pipenv install [package]

#離開環境
ctrl + d

#啟動環境
pipenv shell

```

### 於heruku 指定要執行的py
```python=
heroku run python manage.py deploy

```

- gunicorn


---
## Deploy Flask to Heruku
> run.py
> requirements.txt
> Procfile
> 
```python=
#建立APP
heroku create [app-name]
```
If you haven't already, log in to your Heroku account and follow the prompts to create a new SSH public key.

$ heroku login
Create a new Git repository
Initialize a git repository in a new or existing directory
```python=

$ cd my-project/
$ git init
$ heroku git:remote -a chattest-aaron
```
Deploy your application
Commit your code to the repository and deploy it to Heroku using Git.

```python=
$ git add .
$ git commit -am "make it better"
$ git push heroku master
```
## Deploy Mongodb to Heroku

> http://blog.kenyang.net/2014/01/22/heroku-getting-started-with-mongodb-and
> https://blog.johlmike.com/2016/08/05/heroku-cha-jian-mlab-mongodb-zi-liao-ku/

```python=
#建立
heroku addons:add mongolab

```

- pymongo.errors.OperationFailure: Authentication failed
```python=
    conn = pymongo.MongoClient('mongodb://aaron:yiyouyop10@ds147723.mlab.com:47723/heroku_rdnsvl95?retryWrites=false')
    #uri = "mongodb://aaron:yiyouyop10@ds147723.mlab.com:47723/heroku_rdnsvl95?authMechanism=SCRAM-SHA-1&retryWrites=false"
    #client = MongoClient(uri)
    # Get the database
    db = conn.heroku_rdnsvl95
    #db.authenticate('aaron','yiyouyop10')
    collection = db.test
    rlts = collection.find()
    collection.insert_one({ "name": "John", "address": "Highway 37" })  
    for row in rlts:
        test = row
    return row['name']
```

- Insert data
```python=
collection.insert_one({ "name": "John", "address": "Highway 37" })  
    
for x in contents:
  if x['date2']> latest['date2']:
     print(x['date2'].strftime('%b %d %H:%M:%S %Y'))
     print(latest['date2'].strftime('%b %d %H:%M:%S %Y'))
            result = collection.insert([x]) 
     print('新增：'+ x['title'])
  else :
     print('no：'+ x['title'])
```

- cron job : Flask_APScheduler
```python=
from flask_apscheduler import APScheduler
from flask import Flask


class Config(object):
    JOBS=[
        {
            'id':'job1',
            'func':'__main__:job_1', ##放到heroku時，須將__main__ 改為檔案名稱
            'args':(1,2),
            'trigger':'cron',
            'hour':17,
            'minute':8
        },
        {
            'id':'job2',
            'func':'__main__:job_1',
            'args':(3,4),
            'trigger':'interval',
            'seconds':5
        }
    ]
def job_1(a,b):   # 一個函式，用來做定時任務的任務。
    print(str(a)+' '+str(b))

app=Flask(__name__) # 例項化flask

app.config.from_object(Config())# 為例項化的flask引入配置

@app.route('/')  # 首頁路由
def hello_world():
    return 'hello'


if __name__=='__main__':
    scheduler=APScheduler()  # 例項化APScheduler，放到heroku要時需要放到__main__外面
    scheduler.init_app(app)  # 把任務列表放進flask，放到heroku要時需要放到__main__外面
    scheduler.start() # 啟動任務列表，放到heroku要時需要放到__main__外面
    app.run()  # 啟動flask

```


### Debug

- 無法安裝bs4

> 最後將python 指定為3.7.0 後解決
