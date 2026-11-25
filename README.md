# Django Polls App (SUTT Task 1.5)

This is my submission for SUTT Backend Task 1.5.

I followed the official Django tutorial ("Writing your first Django app") from Part 1 to the end and built the polls app. You can see a list of questions, open one, vote for a choice and then see the results. There is also an admin page where you can add and edit questions.

## How to run it

You need Python installed.

```
git clone https://github.com/Vaibh-1/SUTT_Backend_Task1.5
cd SUTT_Backend_Task1.5
pip install django
python manage.py migrate
python manage.py runserver
```

You also need the Debug Toolbar which can be installed by this:

```
python -m pip install django django-debug-toolbar
python manage.py migrate
python manage.py runserver
```

Run these from the folder that has `manage.py` in it. Then open http://127.0.0.1:8000/polls/ in the browser.

To use the admin page (http://127.0.0.1:8000/admin/), first make a user with `python manage.py createsuperuser`.

To run the tests: `python manage.py test polls`

## How the web works 

### 1. What happens when you type a URL and press Enter?

Say I type `https://example.com/polls/` and press Enter.

1. DNS lookup: The browser cant directly connect to the url it needs an IP adress. So the browser asks a DNS server and it gives back the IP (from the domain's A record). The browser also caches this so it doesn't have to ask every time you want to go to a website.
2. Connection: The browser connects to that IP using TCP, on port 443 since it's https. There is also a TLS handshake so everything sent after this is encrypted.
3. Request: The browser sends an HTTP request, something like GET,POST,etc.
4. Server side: On a real deployment (like AWS), the request first goes through the firewall (security group), which only lets in allowed ports like 443. Then it reaches the server where Nginx (example of a reverse proxy) is running, and Nginx forwards it to the actual app running on a local port like localhost:8000.
5. Response: The app processes the request and sends back a response with a status code, headers and the HTML.
6. Showing the page: The browser reads the HTML,CSS and JS and then it uses it to show us the web page

### 2. What are the client and the server?

The client is whatever is asking for something, usually the browser (it could also be a mobile app) it requests the server for data or change in data. The server is a computer somewhere that has the website's code and data, and its job is to wait for requests and answer them, it recieves requests from the client and work upon the requests.

They talk using HTTP over a TCP connection. The client always starts: it sends a request and the server sends back a response. In normal HTTP the server can't just send stuff to the client on its own.

### 3. What is HTTP? What's inside a request and a response?

HTTP (HyperText Transfer Protocol) is basically a set of rules for how the client and server talk to each other. It follows a request -> response model.

HTTP is also stateless, so the server doesn't remember anything about previous requests. Every request has to carry all the info the server needs (like cookies or authorization info which helps the server to identify the user).


Methods:
It just gives the intent of the interaction which basically in one word tells us that what type of interaction is taking place in between the user and the server
ex = GET(get data) POST(send data) PUT(it also update data but it does that completely) PATCH(update data) DELETE(delete something from the server) OPTIONS(used to fetch the capabilities of the server)


A request has:

- a method (GET, POST, etc.) which tells the intent
- the path (like /polls/1/)
- headers, which give extra info about the request (Host, User-Agent, Accept, Cookie...)
- a body (optional), for example the form data when you vote

A response has:

- a status code (like `200 OK`)
- headers (Content-Type, Set-Cookie, Cache-Control...)
- a body, which is the actual content (HTML, JSON, etc.)

Something like this:

GET /polls/ HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 ...
Accept: text/html

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8



### 4. GET vs POST

GET is for getting data and it shouldn't change anything on the server. Any data goes in the URL (like `/search?q=django`), so it shows up in history and can be bookmarked. GET is idempotent, which means sending the same request many times has the same effect as sending it once.

POST is for sending data to the server, usually to create or change something. The data goes in the request body, not the URL. It's not idempotent, so sending it twice can do the thing twice.

In the polls app, opening a question page is a GET, and submitting the vote form is a POST because it changes the vote count in the database. Getting the results of the poll is also a GET request.

### 5. What do 200, 302, 404 and 500 mean?

The status code tells the client what happened with the request. The first digit gives the category: 2xx is success, 3xx is redirect, 4xx means the client made a mistake and 5xx means the server messed up.

- 200 OK: everything worked and you got what you asked for.
- 302 Found: a temporary redirect. The response has a `Location` header and the browser automatically goes to that URL. In the polls app, after you vote, the view returns an `HttpResponseRedirect` to the results page, which is a 302. The tutorial does this so the vote doesn't get submitted again if you press back or refresh.This prevents accidental resubmission of the form becuase user is redirected to the results page directly.
- 404 Not Found: the thing you asked for doesn't exist. For example `/polls/999/` gives a 404 because there is no question with that id.
- 500 Internal Server Error: something broke in the server code, like a bug or an error that wasn't handled.

### 6. What do HTML, CSS and JavaScript each do?

- HTML is the structure and content of the page: headings, text, links, forms, buttons. Basically the skeleton.It gives the base structure of the website.
- CSS controls how it looks: colors, fonts, spacing, layout. It defines the looks of the buttons the headings and changes the background,it also adds simple basic keyframe animations like fade or loading icon. In Part 6 of the tutorial I added a `style.css` that made the question links green and added a background image.
- JavaScript adds behaviour. It runs in the browser and makes the page interactive, like reacting to clicks or updating part of the page without reloading. JavaScript is for animations that are complex or depend on logic. That includes things triggered by scroll position, user input or data, and games and physics.It also checks input before it is submitted

### 7. How does a website remember who you are between pages?

Since HTTP is stateless, the server would forget you after every request. Cookies fix this.

A cookie is a small piece of data that the server asks the browser to store. The browser then sends it back automatically with every request to that site. This helps the server to understand who the user is and validate the request.

This is how login works:

1. You log in with your username and password.
2. The server checks them.
3. The server sends back a Set-Cookie header with a session ID.
4. The browser stores the cookie.
5. On the next requests, the browser sends it back in the Cookie header.
6. The server looks up the session ID and knows it's you.

The session is the data stored on the server (like which user is logged in), and the cookie only holds the session ID, kind of like a token number. Django does this by default: it saves sessions in the database and the cookie is called `sessionid`. This is how the Django admin keeps you logged in.

### 8. How does a request travel through Django?


Browser -> urls.py -> view -> model (database) -> template -> response -> Browser


Example: I open /polls/1/.

1. URLs: Django checks `mysite/urls.py` first. `polls/` matches, so the rest (`1/`) is passed on to `polls/urls.py` because of `include()`. There `<int:pk>/` matches, so Django knows which view to call and passes 1 as the question id.
2. View: The view is the function (or class) that handles the request. Here it's the detail view.
3. Model: The view gets the data through the model, like `Question.objects.get(pk=1)`. A model is a Python class that maps to a database table, so I don't have to write SQL myself (Django's ORM does it). If the question doesn't exist, it returns a 404.
4. Template: The view passes the question to `polls/detail.html`. The template is HTML with placeholders like `{{ question.question_text }}` and Django fills them in.
5. Response: The final HTML is sent back to the browser as an `HttpResponse`.

In the final version of the tutorial the views are generic views (`ListView`, `DetailView`), which do the "get object or 404" and render part for me, but the flow is the same.

Before the view runs, the request also passes through middleware, which is where things like sessions and the CSRF check happen.

This matches what I learnt about routing: Nginx decides which app gets the request, then the app's router (here `urls.py`) decides which function handles it.

### 9. Why did the tutorial add {% csrf_token %} to the voting form?

CSRF stands for Cross Site Request Forgery.

The problem: say I'm logged in to a site, so my browser has its cookie. If I then visit some evil website, it could have a hidden form that sends a POST request to the real site. My browser would automatically attach my cookie, so the real site would think I sent it.

`{% csrf_token %}` adds a hidden input to the form with a random secret token. When the form is submitted, Django's CSRF middleware checks that the token is there and correct. The evil site can't know this token, so its fake request gets rejected with a 403 Forbidden.

The voting form uses POST and changes data (the vote count), so it needs this. The tutorial says every POST form that submits to your own site should have it.

