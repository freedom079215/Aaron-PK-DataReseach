---
layout: post
title: Chatbot with Dialogflow
date: 2019-10-01
excerpt: ""
tags: NLP, Chatbot, AI, LINE, Dialogflow
comments: true
---
# Chatbot with Dialogflow

1. Architecture
![](https://i.imgur.com/TYnE40J.png)

![](https://i.imgur.com/NRKGH7e.png)

![](https://i.imgur.com/Uyr0POQ.png)


## Webhook

> Webhook是「用戶定義的HTTP回呼」。[2]Webhook通常被某些事件啟用，比如將程式碼推播到版本庫[3]或評論部落格。[4]當此事件發生時，原網站將向為Webhook組態的URL傳送HTTP請求。用戶可組態它們引發網頁上的事件以呼叫另一個網站的行為。此操作可為任何事件。Webhook常用於啟用持續整合系統的構建操作[5]或用於提醒缺陷跟蹤管理系統。[6]由於Webhook使用HTTP，它們可以被無縫整合入網頁服務而無需添加新的基礎設施。[7]但是，除使用HTTP外也有方法構建一個訊息佇列服務，如包括IronMQ和RestMS在內的一些RESTful軟體。

## 運用Flask 建立 Webhook

```
@app.route("/")
```
> https://lvraikkonen.github.io/2017/07/26/Flask%20%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%832:%20%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3@app.route()/

> https://www.pragnakalp.com/dialogflow-fulfillment-webhook-tutorial/

## 測試 chatbot

```python=
ngrok http 5000
```

## Dialogflow Fulfillment

- fullfillmentmessage
> https://cloud.google.com/dialogflow/docs/fulfillment-how?hl=zh-tw
```python=
      "fulfillmentText": "This is a text response",
      "fulfillmentMessages": [
        {
          "card": {
            "title": action['moviename'],
            "subtitle": "card text",
            "formatted_text" :"long texttttttt",
            "imageUri": rsp[1],
            "buttons": [
              {
                "text": "電影介紹網址",
                "postback": rsp[2]['homepage']
              }
            ]
          }
        }
      ]
    }
```

- use "payload" as the format for specifi platform

> https://developers.line.biz/en/reference/messaging-api/#buttons
> 

```python=
   "fulfillmentMessages": [
      {
        "payload": {
          "line": {
            "type": "template",
            "template": {
              "imageBackgroundColor": "#FFFFFF",
              "imageSize": "contain",
              "imageAspectRatio": "rectangle",
              "defaultAction": {
                "label": "Description",
                "type": "uri",
                "uri": rsp[2]['homepage']
              },
              "actions": [
                {
                  "label": "Description",
                  "type": "uri",
                  "uri": rsp[2]['homepage']
                }
              ],
              "text": des,
              "type": "buttons",
              "thumbnailImageUrl": rsp[1],
              "title": action['moviename']
            },
            "altText": "This is a buttons template"
          }
        },
        "platform": "LINE"
      }
    ]
```

## 對話流程設計
> https://ithelp.ithome.com.tw/articles/10203028
> 

* 先在Context的output設定好要變數名稱，左邊數字為存活時間長度
* 在希望做後續回應的Intent, 設定Input Context並且將要設定的變數Parameter, set as a Default Value:
```
#[context_var].parameter
```