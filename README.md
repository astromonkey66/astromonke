<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>AstroMonkey</title>
  <style>
    body {
      font-family: sans-serif;
      background-color: #0b0c10;
      color: #ffffff;
      padding: 1rem;
    }
    header {
      text-align: center;
      background-color: #1f2833;
      padding: 1rem;
      margin-bottom: 2rem;
    }
    h1 { color: #66fcf1; }
    input, textarea {
      width: 100%;
      max-width: 500px;
      padding: 0.5rem;
      margin: 0.5rem 0;
    }
    button {
      background-color: #45a29e;
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      margin: 0.5rem 0;
      cursor: pointer;
    }
    .post {
      background: #1f2833;
      padding: 1rem;
      margin: 1rem 0;
      border-left: 4px solid #66fcf1;
    }
  </style>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
</head>
<body>
  <header>
    <h1>AstroMonker</h1>
    <p>Teile deine astronomischen Ideen mit dem Universum</p>
  </header>

  <section>
    <h2>Neue Idee posten</h2>
    <input type="text" id="name" placeholder="Dein Name oder Nickname (optional)" />
    <textarea id="postText" placeholder="Deine Idee..."></textarea>
    <button onclick="submitPost()">Veröffentlichen</button>
  </section>

  <section>
    <button onclick="togglePosts()">Alle Ideen anzeigen</button>
    <div id="post-list" style="display: none;"></div>
  </section>

  <script>
    // Firebase-Konfiguration – HIER DEINE EINTRAGEN
    const firebaseConfig = {
      apiKey: "DEIN_API_KEY",
      authDomain: "DEIN_AUTH_DOMAIN",
      projectId: "DEIN_PROJECT_ID",
      storageBucket: "DEIN_BUCKET",
      messagingSenderId: "DEIN_SENDER_ID",
      appId: "DEINE_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    function submitPost() {
      const name = document.getElementById("name").value || "Unbekannt";
      const text = document.getElementById("postText").value.trim();
      if (text !== "") {
        db.collection("posts").add({
          name: name,
          text: text,
          timestamp: firebase.firestore.FieldValue.serverTimestamp()
        });
        document.getElementById("postText").value = "";
        document.getElementById("name").value = "";
        alert("Deine Idee wurde gespeichert!");
      }
    }

    let postsVisible = false;

    function togglePosts() {
      const list = document.getElementById("post-list");
      if (!postsVisible) {
        db.collection("posts").orderBy("timestamp", "desc").get().then(snapshot => {
          list.innerHTML = "";
          snapshot.forEach(doc => {
            const post = doc.data();
            const div = document.createElement("div");
            div.className = "post";
            div.innerHTML = `<strong>${post.name}</strong><p>${post.text}</p>`;
            list.appendChild(div);
          });
          list.style.display = "block";
          postsVisible = true;
        });
      } else {
        list.style.display = "none";
        postsVisible = false;
      }
    }
  </script>
</body>
</html>
