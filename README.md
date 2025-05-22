# astromonke
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AstroMonker</title>
  <style>
    body {
      font-family: sans-serif;
      background-color: #0b0c10;
      color: #ffffff;
      padding: 1rem;
    }
    header {
      background-color: #1f2833;
      padding: 1rem;
      text-align: center;
    }
    h1 { color: #66fcf1; }
    input, textarea {
      display: block;
      margin: 0.5rem 0;
      padding: 0.5rem;
      width: 100%;
      max-width: 400px;
    }
    button {
      background-color: #45a29e;
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      cursor: pointer;
    }
    .post {
      background: #1f2833;
      padding: 1rem;
      margin: 1rem 0;
      border-left: 3px solid #66fcf1;
    }
  </style>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
</head>
<body>
  <header>
    <h1>AstroMonker</h1>
  </header>

  <section id="auth">
    <h2>Anmelden / Registrieren</h2>
    <input type="email" id="email" placeholder="E-Mail" />
    <input type="password" id="password" placeholder="Passwort" />
    <button onclick="login()">Anmelden</button>
    <button onclick="register()">Registrieren</button>
    <button onclick="logout()" id="logout" style="display:none;">Abmelden</button>
  </section>

  <section id="new-post" style="display:none;">
    <h2>Neue Idee posten</h2>
    <textarea id="postText" placeholder="Deine astronomische Idee..."></textarea>
    <button onclick="submitPost()">Veröffentlichen</button>
  </section>

  <section id="posts">
    <h2>Ideen aus dem Universum</h2>
    <div id="post-list"></div>
  </section>

  <script>
    // Firebase config – DEINE DATEN HIER EINFÜGEN
    const firebaseConfig = {
      apiKey: "DEIN_API_KEY",
      authDomain: "DEIN_AUTH_DOMAIN",
      projectId: "DEIN_PROJECT_ID",
      storageBucket: "DEIN_STORAGE_BUCKET",
      messagingSenderId: "DEIN_SENDER_ID",
      appId: "DEINE_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const db = firebase.firestore();

    const email = document.getElementById("email");
    const password = document.getElementById("password");
    const logoutBtn = document.getElementById("logout");
    const newPost = document.getElementById("new-post");

    function login() {
      auth.signInWithEmailAndPassword(email.value, password.value)
        .catch(e => alert("Fehler beim Anmelden: " + e.message));
    }

    function register() {
      auth.createUserWithEmailAndPassword(email.value, password.value)
        .catch(e => alert("Fehler bei Registrierung: " + e.message));
    }

    function logout() {
      auth.signOut();
    }

    auth.onAuthStateChanged(user => {
      if (user) {
        logoutBtn.style.display = "inline-block";
        newPost.style.display = "block";
      } else {
        logoutBtn.style.display = "none";
        newPost.style.display = "none";
      }
    });

    function submitPost() {
      const text = document.getElementById("postText").value;
      if (text.trim() !== "") {
        db.collection("posts").add({
          author: auth.currentUser.email,
          text: text,
          timestamp: firebase.firestore.FieldValue.serverTimestamp()
        });
        document.getElementById("postText").value = "";
      }
    }

    function renderPosts() {
      db.collection("posts").orderBy("timestamp", "desc").onSnapshot(snapshot => {
        const list = document.getElementById("post-list");
        list.innerHTML = "";
        snapshot.forEach(doc => {
          const post = doc.data();
          const div = document.createElement("div");
          div.className = "post";
          div.innerHTML = `<strong>${post.author}:</strong><p>${post.text}</p>`;
          list.appendChild(div);
        });
      });
    }

    renderPosts();
  </script>
</body>
</html>
