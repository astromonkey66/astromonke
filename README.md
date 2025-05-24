from flask import Flask, request, render_template_string, redirect
import json, os
from hashlib import sha256

app = Flask(__name__)
USERS_FILE = "users.json"
POSTS_FILE = "posts.json"

def load(filename):
    if not os.path.exists(filename):
        with open(filename, "w") as f: json.dump({}, f)
    with open(filename) as f: return json.load(f)

def save(filename, data):
    with open(filename, "w") as f: json.dump(data, f, indent=2)

@app.route('/')
def home():
    posts = load(POSTS_FILE)
    return render_template_string(TEMPLATE, posts=posts)

@app.route('/register', methods=['POST'])
def register():
    users = load(USERS_FILE)
    u, p = request.form['username'], sha256(request.form['password'].encode()).hexdigest()
    if u in users: return "Benutzername existiert bereits!", 400
    users[u] = {"password": p}
    save(USERS_FILE, users)
    return redirect('/')

@app.route('/post', methods=['POST'])
def post():
    users, posts = load(USERS_FILE), load(POSTS_FILE)
    u, p, t = request.form['
