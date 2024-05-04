---
title: "Hacking a CTF: Sessions aren't safe"
layout: post
author: Zola Gonano
series: CTFs
---

I was playing a CTF at [247CTF.com](https://247ctf.com), called "Secure Session" with the description, "If you can guess our random secret key, we will tell you the flag securely stored in your session." It was surprisingly easy to capture the flag; it took less than a minute. And that's because sessions aren't secure storage for sensitive data.

The code was a simple [Flask](https://palletsprojects.com/p/flask/) app:

```python
import os
from flask import Flask, request, session
from flag import flag

app = Flask(__name__)
app.config['SECRET_KEY'] = os.urandom(24)

def secret_key_to_int(s):
    try:
        secret_key = int(s)
    except ValueError:
        secret_key = 0
    return secret_key

@app.route("/flag")
def index():
    secret_key = secret_key_to_int(request.args['secret_key']) if 'secret_key' in request.args else None
    session['flag'] = flag
    if secret_key == app.config['SECRET_KEY']:
      return session['flag']
    else:
      return "Incorrect secret key!"

@app.route('/')
def source():
    return "

%s

" % open(__file__).read()

if __name__ == "__main__":
    app.run()
```

When I read the index function, I immediately hit Shift + F9 on my Firefox and opened Firefox's storage inspector. There it was: the session with the flag encoded in base64.

```
{"flag":{" b":"MjQ3Q1RGe2RhODA3OTVmOGE1Y2FiMmUwMzdkNzM4NTgwN2I5YTkxfQ=="}}6s1_~';t]Q
```

Then I decoded the flag and got the flag in plain text just like that.

```
247CTF{da80795f8a5cab2e037d7385807b9a91}
```

## But how it could be prevented?

A common method to prevent such problems is to store an identifying key in the session instead of the actual data. The actual data is then securely stored in a database on your server. You retrieve the identifying key (which should be unique and random, such as a [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)), look up the corresponding data from your server, and serve it to the user.

Also, there is something called "**Encrypted Cookies**", which involves encrypting the data before storing it in cookies or sessions on the user's client. There's a Python library for this called `itsdangerous`, which can be used to encrypt and sign session and cookie data securely.

---

My blog and all of its content are available under the `CC by SA 4.0` License on my [GitHub](https://github.com/zolagonano/zolagonano.github.io). If you notice any problems or have any improvements for the blog or its content, you're always welcome to open a pull request.
