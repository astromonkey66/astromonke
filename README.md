from flask import Flask, request, render_template_string, redirect
import json, os
from hashlib import sha256

app = Flask(__name__)
USERS_FILE = "users.json"
POSTS_FILE = "posts.json"

def load(file):
    if not os.path.exists(file): json.dump({}, open(file, "w"))
    return json.load(open(file))

def save(file, data):
    json.dump(data, open(file, "w"), indent=2)

@app.route('/')
def home():
    posts = load(POSTS_FILE)
    return render_template_string(TEMPLATE, posts=posts)

@app.route('/register', methods=['POST'])
def register():
    users = load(USERS_FILE)
    u = request.form['username']
    p = sha256(request.form['password'].encode()).hexdigest()
    if u in users: return "Benutzername existiert bereits!", 400
    users[u] = {"password": p}
    save(USERS_FILE, users)
    return redirect('/')

@app.route('/post', methods=['POST'])
def post():
    users, posts = load(USERS_FILE), load(POSTS_FILE)
    u, p = request.form['username'], sha256(request.form['password'].encode()).hexdigest()
    t = request.form['text']
    if u not in users or users[u]['password'] != p: return "Ungültige Anmeldung", 403
    post_id = str(len(posts) + 1)
    posts[post_id] = {"author": u, "text": t, "comments": []}
    save(POSTS_FILE, posts)
    return redirect('/')

@app.route('/comment/<pid>', methods=['POST'])
def comment(pid):
    posts = load(POSTS_FILE)
    if pid not in posts: return "Post nicht gefunden", 404
    posts[pid]["comments"].append({
        "user": request.form['username'],
        "comment": request.form['comment']
    })
    save(POSTS_FILE, posts)
    return redirect('/')

TEMPLATE = '''
<!DOCTYPE html>
<html>
<head>
  <title>Astromonkey – Theorien teilen</title>
</head>
<body>
  <h1>Astromonkey</h1>

  <h2>Registrieren</h2>
  <form action="/register" method="post">
    Nutzername: <input name="username" required>
    Passwort: <input name="password" type="password" required>
    <button type="submit">Registrieren</button>
  </form>

  <h2>Theorie posten</h2>
  <form action="/post" method="post">
    Nutzername: <input name="username" required>
    Passwort: <input name="password" type="password" required><br>
    Theorie: <textarea name="text" required></textarea>
    <button type="submit">Posten</button>
  </form>

  <h2>Alle Theorien</h2>
  {% for id, post in posts.items() %}
    <div style="border:1px solid #000; padding:10px; margin:10px;">
      <b>{{ post.author }}</b> schrieb:<br>
      <p>{{ post.text }}</p>

      <h4>Kommentare:</h4>
      <ul>
        {% for c in post.comments %}
          <li><b>{{ c.user }}</b>: {{ c.comment }}</li>
        {% endfor %}
      </ul>

      <form action="/comment/{{ id }}" method="post">
        Nutzername: <input name="username" required>
        Kommentar: <input name="comment" required>
        <button type="submit">Kommentieren</button>
      </form>
    </div>
  {% endfor %}
</body>
</html>
'''

if __name__ == '__main__':
    app.run()
