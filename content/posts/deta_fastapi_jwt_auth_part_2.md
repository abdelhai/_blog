---
title: "Get started with FastAPI JWT authentication – Part 2"
date: 2021-04-22
draft: false
---
# Get started with FastAPI JWT authentication – Part 2
This is the second of a two part series on implementing authorization in a FastAPI application using Deta. In the previous article, we learned a bit about JWT, set up the project, and finished the building blocks of authorization logic. In this article, let's implement the logic, and deploy our app on Deta micros! [The full code is available here.](https://github.com/rohanshiva/Deta-FastAPI-JWT-Auth-Blog) 

## Implementing the auth logic

Before we implement the auth logic, let's create a data model for login and signup details. 

In `user_model.py` :

```python
from pydantic import BaseModel

class AuthModel(BaseModel):
    username: str
    password: str
```

This model represents the data that we can expect from the client when they hit `/login` or `/signup` endpoints.

Update the `main.py` , with the following import statements

```python
from auth import Auth
from user_model import AuthModel
from fastapi import FastAPI, HTTPException, Security
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
```

Also, we will create an `auth_handler` to access the logic from the `Auth` class. We will use `security` in our protected endpoints to access the token from the request header.

```python
security = HTTPBearer()
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
def login(user_details: AuthModel):
    user = users_db.get(user_details.username)
    if (user is None):
        return HTTPException(status_code=401, detail='Invalid username')
    if (not auth_handler.verify_password(user_details.password, user['password'])):
        return HTTPException(status_code=401, detail='Invalid password')
    
    access_token = auth_handler.encode_token(user['key'])
    refresh_token = auth_handler.encode_refresh_token(user['key'])
    return {'access_token': access_token, 'refresh_token': refresh_token}
```

If the account with the username doesn't exist, or if the hashed password in the `users_db` doesn't match the input password we can simply raise an `HTTPException`. Otherwise, we can return `access_token` and `refresh_token`.

```python
@app.post('/secret')
def secret_data(credentials: HTTPAuthorizationCredentials = Security(security)):
    token = credentials.credentials
    if(auth_handler.decode_token(token)):
        return 'Top Secret data only authorized users can access this info'

@app.get('/notsecret')
def not_secret_data():
    return 'Not secret data'
```

The `/secret` endpoint only returns the "Secret Data" if the token argument is valid. However, if the token is invalid or an expired token, then `decode_token` raises a `HTTPException`. The token is usually passed in the request header as `Authorization: Bearer <token>`. Therefore, to get the token we can wrap the input `credentials` around `HTTPAuthorizationCredentials` tag. Now we can access the token from the request header in `credentials.credentials`. 

The `/not_secret` endpoint is an example of an unprotected endpoint, which doesn't require any authentication.

```python
@app.get('/refresh_token')
def refresh_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    refresh_token = credentials.credentials
    new_token = auth_handler.refresh_token(refresh_token)
    return {'access_token': new_token}
```

`/refresh_token` endpoint is also pretty simple, it receives a refresh token which is then passed onto the the function from auth logic to get a new token. 

Here is a look at `main.py` at the end:

```python
from fastapi import FastAPI, HTTPException, Security
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from auth import Auth
from deta import Deta
from user_model import AuthModel

deta = Deta()
users_db = deta.Base('users')

app = FastAPI()

security = HTTPBearer()
auth_handler = Auth()

@app.post('/signup')
def signup(user_details: AuthModel):
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
def login(user_details: AuthModel):
    user = users_db.get(user_details.username)
    if (user is None):
        return HTTPException(status_code=401, detail='Invalid username')
    if (not auth_handler.verify_password(user_details.password, user['password'])):
        return HTTPException(status_code=401, detail='Invalid password')
    
    access_token = auth_handler.encode_token(user['key'])
    refresh_token = auth_handler.encode_refresh_token(user['key'])
    return {'access_token': access_token, 'refresh_token': refresh_token}

@app.get('/refresh_token')
def refresh_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    refresh_token = credentials.credentials
    new_token = auth_handler.refresh_token(refresh_token)
    return {'access_token': new_token}

@app.post('/secret')
def secret_data(credentials: HTTPAuthorizationCredentials = Security(security)):
    token = credentials.credentials
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
  "password": "$2b$12$Ml66gTN0j4sAOkoDJ1aKnOmP0ye1FBNNk1QzEt0/6LgemgOUj469e"
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
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTkwMTY4MzgsImlhdCI6MTYxOTAxNTAzOCwic3ViIjoiZmx5aW5nc3BvbmdlIn0.XnPaDwmj30M3vMOaPUOrqESIBNx0mctbjNW5jY8hIjQ",
  "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTkwNTEwMzgsImlhdCI6MTYxOTAxNTAzOCwic3ViIjoiZmx5aW5nc3BvbmdlIn0.Pm68HQFM6NwUxnEjFUxxTiOYT3KBqchD_e2g0LfP5Co"
}
```

`/secret`

```python
curl -X 'POST' \
  'http://127.0.0.1:8000/secret' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTkwMTY2MTIsImlhdCI6MTYxOTAxNDgxMiwic3ViIjoicmFta2kifQ.U8fUN1x1Gz18f7oA_y7Z9DE-1FIH9Ps0sDilwORvmI8' \
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
  'http://127.0.0.1:8000/secret' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTg5MzU3MzcsImlhdCI6MTYxODkzNTY3Nywic3ViIjoicm9oYW4ifQ.dja0E6SUaZfEvYVKySjLE9OLXOtob5pjpy3R_rlCD7c' \
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
  'http://127.0.0.1:8000/refresh_token' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTkwNTA3NzYsImlhdCI6MTYxOTAxNDc3Niwic3ViIjoicmFta2kifQ.2J0O4RKwEcABxe6hEX7ZshMo66J6D0dD6-hYeFLBFGg'
Response Body
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MTkwMTY2MTIsImlhdCI6MTYxOTAxNDgxMiwic3ViIjoicmFta2kifQ.U8fUN1x1Gz18f7oA_y7Z9DE-1FIH9Ps0sDilwORvmI8"
}
```

## Deploy on Deta micros
Before we begin, make sure to [install the Deta CLI.](https://docs.deta.sh/docs/cli/install) After installing, run the following commands in the same directory to deploy our app on Deta micros.

```json
deta login
```
We also need to add a `.env` file with the secret.

```
APP_SECRET_STRING=SECRET_STRING
```

Now we need to update our micro by doing:

```python
deta new 
deta update -e .env
deta deploy
```



## Summary

Our simple FastAPI application with JWT auth is now ready! As you can probably tell, we are not doing anything "secret" with our authorization. This article is just a template for implementing authorization. You can build on this template to build a fullstack application that relies on authorization. [The full code is available here.](https://github.com/rohanshiva/Deta-FastAPI-JWT-Auth-Blog)
