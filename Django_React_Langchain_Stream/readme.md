# Installations

- pip install django
- django-admin startproject Django_React_Langchain_Stream
- cd Django_React_Langchain_Stream

# Create Your Django App

- pip install Django==5.0.3
- python manage.py startapp langchain_stream

#  Frontend Setup with React

> node : v20.8.0
> npm  : 10.1.0

# Create a React Application

- npm create vite@latest
** Name the project frontend ** 
- cd frontend
- npm install

# Test the Frontend

- npm run dev
** After completing the setup and installations, your project directory should look like this: **

        Django_React_Langchain_Stream/
        ├── Django_React_Langchain_Stream/
        ├── frontend/
        ├── langchain_stream/
        ├── venv/
        ├── db.sqlite3
        └── manage.py

# Bridging Django and React with Websockets

- OPENAI_API_KEY=this-is-a-fake-api-key-replace-it

# Configure the Django settings.py for Websockets
- In settings.py, add langchain_stream and daphne to INSTALLED_APPS:

```python
'daphne',
# ...,
'langchain_stream',
#
# WSGI_APPLICATION = ' Django_React_Langchain_Stream.wsgi.application'
ASGI_APPLICATION = "Django_React_Langchain_Stream.asgi.application"
```
# Create the views.py file

> pip install langchain==0.1.11 langchain-community==0.0.26 langchain-openai==0.0.8 channels==4.0.0 daphne==4.1.0 python-dotenv==1.0.1

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from channels.generic.websocket import AsyncWebsocketConsumer
import json
from dotenv import load_dotenv

load_dotenv('.env')

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")

output_parser = StrOutputParser()
# Chain
chain = prompt | llm.with_config({"run_name": "model"}) | output_parser.with_config({"run_name": "Assistant"})


class ChatConsumer(AsyncWebsocketConsumer):

    async def connect(self):
        await self.accept()

    async def disconnect(self, close_code):
        pass

    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json["message"]

        try:
            # Stream the response
            async for chunk in chain.astream_events({'input': message}, version="v1", include_names=["Assistant"]):
                if chunk["event"] in ["on_parser_start", "on_parser_stream"]:
                    await self.send(text_data=json.dumps(chunk))

        except Exception as e:
            print(e)
```

# Set Up Websocket Routing

- Create the file: langchain_stream/routing.py,and add the following code:
```python
from django.urls import re_path  
from . import views  
  
websocket_urlpatterns = [  
    re_path(r'ws/chat/$', views.ChatConsumer.as_asgi()),  
]

```
# Create the file: langchain_stream/urls.py,and add the following code:

```python
from django.urls import path  
from . import views  
  
  
urlpatterns = [  
    path('ws/chat/', views.ChatConsumer.as_asgi()),  
]

```

- Replace the code in Django_React_Langchain_Stream/asgi.py with the following:

```python
import os  
from django.core.asgi import get_asgi_application  
from channels.routing import ProtocolTypeRouter, URLRouter  
from channels.auth import AuthMiddlewareStack  
import langchain_stream.routing  
  
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Django_React_Langchain_Stream.settings')  
  
application = ProtocolTypeRouter({  
  "http": get_asgi_application(),  
  "websocket": AuthMiddlewareStack(  
        URLRouter(  
            langchain_stream.routing.websocket_urlpatterns  
        )  
    ),  
})

```

# Integrate with React

- App.jsx

# Add CSS Styling

- App.css

# run 

- python manage.py runserver
- cd frontend
- npm run dev

# git

- git add .
- git commit -m " firt commit"
- git remote add origin <url>
- git rebase origin/main
- git push --set-upstream origin main

*** 
add OPENAI_API_KEY="" in .env