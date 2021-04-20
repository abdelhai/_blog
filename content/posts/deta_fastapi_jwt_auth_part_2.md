---
title: "Get started with FastAPI JWT authentication – Part 2"
date: 2021-04-13
draft: false
---
# Get started with FastAPI JWT authentication – Part 2
This is the second of a two part series on implementing authorization in a FastAPI application using Deta. In the previous article, we learned a bit about JWT, set up the project, and finished the building blocks of authorization logic. In this article, let's implement the logic, and deploy our app on Deta micros! [The full code is available here.](https://github.com/rohanshiva/Deta-FastAPI-JWT-Auth-Blog) 

## Implementing the auth logic

Before we implement the auth logic, let's create a data modal for login and signup details. 

In `user_modal.py` :

```python
from pydantic import BaseModel

class AuthModal(BaseModel):
    username: str
    password: str
```

This modal represents the data that we can expect from the client when they hit `/login` or `/signup` endpoints.

Update the `main.py` , with the following import statements

```python
from auth import Auth
from user_modal import AuthModal
```

Also, we will create an `auth_handler` to access the logic from the `Auth` class

```python
auth_handler = Auth()
```

Here is the code for `/signup` endpoint. 

```python
@app.post('/signup')
def signup(user_details: AuthModal):
    if users_db.get(user_details.username) != None:
        return 'Account already exists'
    try:
        hashed_password = auth_handler.encode_password(user_details.password)
        user = {'key': user_details.username, 'password': hashed_password}
        return users_db.put(user)
    except:
        error_msg = 'Failed to signup user'
        return error_msg
```

In this function we are checking if a user with the username already exists in our `users_db`. If so, we can simply return a message indicating that the account already exists. If the user doesn't already exists we can hash the password using the `encode_password` function from `auth.py` and store the user in `users_db`. In case of any errors while adding the user to the base, we can return a failure message. 

`/login` endpoint is pretty simple. This also takes in the argument `user_details` , which has the username and password. 

```python
@app.post('/login')
def login(user_details: AuthModal):
    user = users_db.get(user_details.username)
    if (user is None):
        return HTTPException(status_code=401, detail='Invalid username')
    if (not auth_handler.verify_password(user_details.password, user['password'])):
        return HTTPException(status_code=401, detail='Invalid password')
    
    token = auth_handler.encode_token(user['key'])
    return {'token': token}
```

If the account with the username doesn't exist, or if the hashed password in the `users_db` doesn't match the input password we can simply raise an `HTTPException`. Otherwise, we can return the encoded JWT token using `encode_token`.

```python
@app.post('/secret')
def secret_data(Authorization: str):
    token = Authorization[8:]
    if(auth_handler.decode_token(token)):
        return 'Top Secret data only authorized users can access this info'

@app.get('/notsecret')
def not_secret_data():
    return 'Not secret data'
```

The `/secret` endpoint only returns the "Secret Data" if the token argument is valid. However, if the token is invalid or an expired token, then `decode_token` raises a `HTTPException`. The argument  `Authorization` is normally in this format `Authorization: Bearer <token>`. Therefore, to get the token we can take the substring from index 8. 

The `/not_secret` endpoint is an example of an unprotected endpoint, which doesn't require any authentication.

```python
@app.get('/refresh_token')
def refresh_token(Authorization: str):
    expired_token = Authorization[8:]
    return auth_handler.refresh_token(expired_token)
```

`/refresh_token` endpoint is also pretty simple, it receives an expired token which is then passed onto the the `refresh_token` function from auth logic to get the new token. 

Here is a look at `main.py` at the end:

```python
from fastapi import FastAPI, HTTPException
from deta import Deta
from auth import Auth
from user_modal import AuthModal

deta = Deta()
users_db = deta.Base('users')

app = FastAPI()

auth_handler = Auth()

@app.post('/signup')
def signup(user_details: AuthModal):
    if users_db.get(user_details.username) != None:
        return 'Account already exists'
    try:
        hashed_password = auth_handler.encode_password(user_details.password)
        user = {'key': user_details.username, 'password': hashed_password}
        return users_db.put(user)
    except:
        error_msg = 'Failed to signup user'
        return error_msg

@app.post('/login')
def login(user_details: AuthModal):
    user = users_db.get(user_details.username)
    if (user is None):
        return HTTPException(status_code=401, detail='Invalid username')
    if (not auth_handler.verify_password(user_details.password, user['password'])):
        return HTTPException(status_code=401, detail='Invalid password')
    
    token = auth_handler.encode_token(user['key'])
    return {'token': token}

@app.get('/refresh_token')
def refresh_token(Authorization: str):
    expired_token = Authorization[8:]
    return auth_handler.refresh_token(expired_token)

@app.post('/secret')
def secret_data(Authorization: str):
    token = Authorization[8:]
    if(auth_handler.decode_token(token)):
        return 'Top Secret data only authorized users can access this info'

@app.get('/notsecret')
def not_secret_data():
    return 'Not secret data'
```

To test the app, go to the terminal in the same directory and run `uvicorn main:app`, you can then go `/docs` on the local endpoint (for me it was [`http://127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs)) to test the application.

`/signup`

```json
curl -X 'POST' \
  'http://127.0.0.1:8000/signup' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "username": "flyingsponge",
  "password": "SpongePassword"
}'

Response Body
{
  "key": "flyingsponge",
  "password": "$2b$12$/Gq7g40zZ4/sQ9iWfqVze.Jx5HI5XwCrERITGG/wZivuZ9jhkd0bK"
}
```

`/login`

```json
curl -X 'POST' \
  'http://127.0.0.1:8000/login' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "username": "flyingsponge",
  "password": "SpongePassword"
}'

Response Body
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgyNDE1MjAsImlhdCI6MTYxODIzOTcyMCwic3ViIjoiZmx5aW5nc3BvbmdlIn0.SoMeSo_b9z4fC-XnR8bepUbFvWvSEw9rRQ9LMJNzm3k"
}
```

`/secret`

```python
curl -X 'POST' \
  'http://127.0.0.1:8000/secret?Authorization=%20Bearer%20eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgyNDE1MjAsImlhdCI6MTYxODIzOTcyMCwic3ViIjoiZmx5aW5nc3BvbmdlIn0.SoMeSo_b9z4fC-XnR8bepUbFvWvSEw9rRQ9LMJNzm3k' \
  -H 'accept: application/json' \
  -d ''

Response body
"Top Secret data only authorized users can access this info"
```

`/notsecret`

```json
curl -X 'GET' \
  'http://127.0.0.1:8000/notsecret' \
  -H 'accept: application/json'

Response Body
"Not secret data"
```

`/secret`

```json
curl -X 'POST' \
  'http://127.0.0.1:8000/secret?Authorization=%20Bearer%20eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMjExMzEsImlhdCI6MTYxODMyMTA3MSwic3ViIjoic2lkIn0.dMPPC95JWeL_rlFfZwgtcVJ9OSi7k6ArZoxOvVbzbZw' \
  -H 'accept: application/json' \
  -d ''

Response Body
{
  "detail": "Token expired"
}
```

Now that the token is expired, let's get a new one

`/refresh_token`

```json
curl -X 'GET' \
  'http://127.0.0.1:8000/refresh_token?Authorization=%20Bearer%20eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMjExMzEsImlhdCI6MTYxODMyMTA3MSwic3ViIjoic2lkIn0.dMPPC95JWeL_rlFfZwgtcVJ9OSi7k6ArZoxOvVbzbZw' \
  -H 'accept: application/json'

Response Body
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTgzMjQ1NjksImlhdCI6MTYxODMyNDUwOSwic3ViIjoic2lkIn0.4RL1t-oa1OOGQoA-AZcgoIoa14J-l-wLHfBtAXX3ik4"
}
```

## Deploy on Deta micros

Run the following commands in the same directory to deploy our app on Deta micros.

```json
deta login
```
We also need to add a `.env` file with the secret.

```
APP_SECRET_STRING=SECRET_STRING
```

Now run the following commands to deploy our app on micros.

```python
deta new 
deta update -e .env
deta deploy
```



## Summary

Our simple FastAPI application with JWT auth is now ready! As you can probably tell, we are not doing anything "secret" with our authorization. This article is just a template for implementing authorization. You can build on this template to build a fullstack application that relies on authorization. [The full code is available here.](https://github.com/rohanshiva/Deta-FastAPI-JWT-Auth-Blog)
