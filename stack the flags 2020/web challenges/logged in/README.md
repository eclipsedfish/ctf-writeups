# Logged In

## Challenge Description
It looks like COViD's mobile application is connecting to this API! Fortunately, our agents stole part of the source code. Can you find a way to log in?

[API Server](http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41061/)

Please view this Document for download instructions.

ZIP File Password: web-challenge-6

Note: Wondering what the second flag is about? Maybe check for a MOBILE Network?
## Solution
Firstly, I downloaded the source code and in it contained a bunch of folders and a JavaScript file.
![1](https://user-images.githubusercontent.com/52084346/101290710-08671300-383f-11eb-9fe4-3c5829e9dd05.PNG)

I opened the app.js file in VS Code and immediately saw the
![2](https://user-images.githubusercontent.com/52084346/101290714-0a30d680-383f-11eb-9b40-3b7c51589e8d.PNG)

```
app.use('/api', apiRouter);
```
 
Which meant that to send requests to the API server you had to add a "/api" after the API server link.

I then digged around more and found the api.js file in routes folder and opened it.
![3](https://user-images.githubusercontent.com/52084346/101290715-0a30d680-383f-11eb-9c24-58d46d58f8f2.PNG)
In the api.js folder was 3 routes:
-/
-/login
-/user/:userId

I opened postman and tried sending a empty POST request to the API server
```
http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41061/api/login
```


and I got back 
![4](https://user-images.githubusercontent.com/52084346/101290716-0ac96d00-383f-11eb-863c-62a573d1a14d.PNG)

```
{"error":"Invalid parameters: username, password"}
```
which meant that the code was checking for those 2 parameters in the POST request and the code was found in validators.js located inside the middlewares folder
![5](https://user-images.githubusercontent.com/52084346/101290825-84f9f180-383f-11eb-900c-e00ab58f2c2f.PNG)
```
 check('username').exists(),
 check('password').exists()
```

So I added those 2 parameters in the body of my POST request in JSON 
```
{
    "username": "",
    "password": ""
}
```


and sent the request and and got the flag:
![6](https://user-images.githubusercontent.com/52084346/101290845-a4911a00-383f-11eb-8e08-a38caca39a87.PNG)
which was flagOne
```
govtech-csg{m!sS1nG_cR3DeN+!@1s}
```


