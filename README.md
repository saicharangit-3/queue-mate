
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>User Dashboard</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: #f4fdf4;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }

    .container {
      background: #fff;
      margin-top: 60px;
      padding: 2rem;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      width: 90%;
      max-width: 500px;
    }

    h2 {
      background: #1976d2;
      color: #fff;
      margin: -2rem -2rem 1.5rem -2rem;
      padding: 15px;
      border-radius: 10px 10px 0 0;
      font-size: 20px;
      text-align: center;
    }

    p {
      margin-bottom: 1rem;
      color: #333;
      text-align: center;
    }

    /* user data will be hidden */
    #userData {
      display: none;
    }

    input, button {
      margin: 0.5rem 0;
      padding: 0.8rem;
      width: 100%;
      border-radius: 6px;
      border: 1px solid #ddd;
      font-size: 16px;
    }

    button {
      border: none;
      background: #1976d2;
      color: #fff;
      cursor: pointer;
      transition: background 0.3s;
    }

    button:hover:enabled {
      background: #0d47a1;
    }

    button:disabled {
      background: #9e9e9e;
      cursor: not-allowed;
    }

    #tokenDisplay {
      margin-top: 15px;
      padding: 10px;
      background: #e8f5e9;
      border: 1px solid #4caf50;
      border-radius: 6px;
      color: #2e7d32;
      font-size: 18px;
      font-weight: bold;
      text-align: center;
      display: none;
    }

    footer {
      margin-top: 20px;
      text-align: center;
      font-size: 12px;
      color: #666;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>User Dashboard</h2>
    <p>Welcome! Checking your data from Firebase...</p>

    <div id="userData"></div>

    <!-- QR Session Input -->
    <input type="text" id="qrIdInput" placeholder="Enter QR session ID">

    <button id="getTokenBtn">Get Token</button>

    <div id="tokenDisplay"></div>

    <footer>©️ DigiToken 2025</footer>
  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-app.js";
    import { 
      getFirestore, collection, query, where, getDocs, 
      addDoc, serverTimestamp, orderBy, limit 
    } from "https://www.gstatic.com/firebasejs/12.2.1/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyC3J_G3alwbNdhoi2A7aSAgVuDhFcG6kLs",
      authDomain: "queuemate-6dded.firebaseapp.com",
      databaseURL: "https://queuemate-6dded-default-rtdb.firebaseio.com",
      projectId: "queuemate-6dded",
      storageBucket: "queuemate-6dded.firebasestorage.app",
      messagingSenderId: "290732340178",
      appId: "1:290732340178:web:62373c241692c27e1a8e0b"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    const mobile = localStorage.getItem("mobile");
    const dept = localStorage.getItem("dept") || "General";
    const branch = localStorage.getItem("branch") || "Main";
    const service = localStorage.getItem("service") || "Default";

    const tokenDisplay = document.getElementById("tokenDisplay");
    const getTokenBtn = document.getElementById("getTokenBtn");

    // Removed showing Firebase user data (userData div is hidden by CSS)

    document.getElementById("getTokenBtn").addEventListener("click", async () => {
      const qrId = document.getElementById("qrIdInput").value.trim();
      if (!mobile || !qrId) {
        alert("❌ Please enter your QR session ID and login.");
        return;
      }

      try {
        const q = query(
          collection(db, "tokens"),
          where("dept", "==", dept),
          where("branch", "==", branch),
          where("service", "==", service),
          where("qrId", "==", qrId),
          orderBy("createdAt", "desc"),
          limit(1)
        );

        const snapshot = await getDocs(q);
        let newNumber = 1;
        if (!snapshot.empty) {
          const lastToken = snapshot.docs[0].data();
          newNumber = (lastToken.number || 0) + 1;
        }

        await addDoc(collection(db, "tokens"), {
          phone: mobile,
          dept,
          branch,
          service,
          qrId,
          number: newNumber,
          createdAt: serverTimestamp(),
          status: "waiting"
        });

        tokenDisplay.style.display = "block";
        tokenDisplay.textContent = `🎟 Your Token Number: ${newNumber}`;

        // Disable button after one token is generated
        getTokenBtn.disabled = true;

        alert(`🎟 Token #${newNumber} generated successfully!`);
      } catch (err) {
        alert("Error generating token: " + err.message);
      }
    });
  </script>
</body>
</html>
