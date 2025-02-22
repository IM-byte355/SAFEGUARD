# SAFEGUARD<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SafeCard - Manage Your Cards</title>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
  <meta http-equiv="permissions-policy" content="identity-credentials-get=(self)">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Arial', sans-serif;
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    .container {
      background: rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(10px);
      border-radius: 15px;
      padding: 30px;
      width: 100%;
      max-width: 500px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    }
    h2 { text-align: center; margin-bottom: 20px; }
    label { display: block; margin-bottom: 8px; font-weight: bold; }
    input, select {
      width: 100%;
      padding: 12px;
      margin-bottom: 15px;
      border: none;
      border-radius: 8px;
      background: rgba(255, 255, 255, 0.2);
      color: #fff;
    }
    button {
      width: 100%; padding: 12px; border: none; border-radius: 8px;
      background: #667eea; color: #fff; font-weight: bold;
      cursor: pointer; transition: background 0.3s ease;
    }
    button:hover { background: #764ba2; }
    .section { margin-top: 20px; }
    .transaction {
      background: rgba(255, 255, 255, 0.1);
      padding: 10px; border-radius: 8px;
      margin-bottom: 10px;
    }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { padding: 8px; text-align: left; border-bottom: 1px solid rgba(255, 255, 255, 0.1); }
    th { background: rgba(255, 255, 255, 0.2); }
  </style>
</head>
<body>
  <div class="container">
    <h2>SafeCard - Manage Your Cards</h2>
    <div id="g_id_onload" data-client_id="YOUR_GOOGLE_CLIENT_ID" data-callback="onSignIn"></div>
    <div class="g_id_signin" data-type="standard"></div>
    <h3>Sign Up / Sign In</h3>
    <input type="text" id="nickname" placeholder="Enter Nickname">
    <input type="email" id="email" placeholder="Enter Email">
    <input type="password" id="password" placeholder="Enter Password">
    <button onclick="signUp()">Sign Up</button>
    <button onclick="signIn()">Sign In</button>
    <label for="cardSelect">Select Card:</label>
    <select id="cardSelect"></select>
    <label for="currencySelect">Select Currency:</label>
    <select id="currencySelect"></select>
    <button onclick="checkBalance()">Check Balance</button>
    <div id="balanceResult"></div>
    <div class="section">
      <h3>Set Spending Limit</h3>
      <input type="number" id="spendingLimit" placeholder="Enter limit">
      <button onclick="setSpendingLimit()">Set Limit</button>
      <div id="spendingLimitResult"></div>
    </div>
    <div class="section">
      <h3>Transaction History</h3>
      <button onclick="loadTransactionHistory()">Load Transactions</button>
      <table id="transactionTable"></table>
    </div>
  </div>
  <script>
    const API_URL = 'http://localhost:5000/api'; // Replace with your backend URL
    const currencies = ["USD", "EUR", "GBP", "INR", "JPY", "AUD", "CAD", "CNY", "CHF", "SEK"];

    // Populate card and currency dropdowns
    document.addEventListener("DOMContentLoaded", async () => {
      const cardSelect = document.getElementById("cardSelect");
      const currencySelect = document.getElementById("currencySelect");

      // Fetch user's cards
      try {
        const response = await fetch(`${API_URL}/cards`, {
          headers: { Authorization: `Bearer ${localStorage.getItem('token')}` },
        });
        const cards = await response.json();
        cards.forEach(card => {
          const option = document.createElement("option");
          option.value = card._id;
          option.textContent = `****-****-****-${card.cardNumber.slice(-4)}`;
          cardSelect.appendChild(option);
        });
      } catch (err) {
        console.error("Failed to fetch cards:", err);
      }

      // Populate currency dropdown
      currencies.forEach(currency => {
        const option = document.createElement("option");
        option.value = currency;
        option.textContent = currency;
        currencySelect.appendChild(option);
      });
    });

    // Google Sign-In callback
    function onSignIn(googleUser) {
      console.log("User signed in:", googleUser);
      alert(`Welcome, ${googleUser.getBasicProfile().getName()}!`);
    }

    // Sign Up
    async function signUp() {
      const nickname = document.getElementById("nickname").value;
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      try {
        const response = await fetch(`${API_URL}/auth/signup`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ nickname, email, password }),
        });
        const data = await response.json();
        if (response.ok) {
          localStorage.setItem('token', data.token);
          alert('Sign-up successful!');
        } else {
          throw new Error(data.error);
        }
      } catch (err) {
        alert(err.message);
      }
    }

    // Sign In
    async function signIn() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      try {
        const response = await fetch(`${API_URL}/auth/signin`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });
        const data = await response.json();
        if (response.ok) {
          localStorage.setItem('token', data.token);
          alert('Sign-in successful!');
        } else {
          throw new Error(data.error);
        }
      } catch (err) {
        alert(err.message);
      }
    }

    // Check Balance
    async function checkBalance() {
      const card = document.getElementById("cardSelect").value;
      const currency = document.getElementById("currencySelect").value;
      try {
        const response = await fetch(`${API_URL}/cards/${card}/balance?currency=${currency}`, {
          headers: { Authorization: `Bearer ${localStorage.getItem('token')}` },
        });
        const data = await response.json();
        if (response.ok) {
          document.getElementById("balanceResult").innerText = `Balance: ${data.balance} ${currency}`;
        } else {
          throw new Error(data.error);
        }
      } catch (err) {
        alert(err.message);
      }
    }

    // Set Spending Limit
    async function setSpendingLimit() {
      const card = document.getElementById("cardSelect").value;
      const limit = parseFloat(document.getElementById("spendingLimit").value);
      try {
        const response = await fetch(`${API_URL}/cards/${card}/limit`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
          body: JSON.stringify({ limit }),
        });
        const data = await response.json();
        if (response.ok) {
          document.getElementById("spendingLimitResult").innerText = `Limit set to ${limit} USD`;
        } else {
          throw new Error(data.error);
        }
      } catch (err) {
        alert(err.message);
      }
    }

    // Load Transaction History
    async function loadTransactionHistory() {
      const card = document.getElementById("cardSelect").value;
      try {
        const response = await fetch(`${API_URL}/cards/${card}/transactions`, {
          headers: { Authorization: `Bearer ${localStorage.getItem('token')}` },
        });
        const data = await response.json();
        if (response.ok) {
          let table = `<tr><th>Date</th><th>Description</th><th>Amount</th></tr>`;
          data.forEach(t => {
            table += `<tr><td>${new Date(t.date).toLocaleDateString()}</td><td>${t.description}</td><td>${t.amount} USD</td></tr>`;
          });
          document.getElementById("transactionTable").innerHTML = table;
        } else {
          throw new Error(data.error);
        }
      } catch (err) {
        alert(err.message);
      }
    }
  </script>
</body>
</html>
