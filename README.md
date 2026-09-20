<!DOCTYPE html>
<html lang="tl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>RMVillasis Enterprises POS</title>
  <style>
    /* --- GENERAL STYLES (Light Blue Theme) --- */
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #e6f2ff; /* Light Blue background */
      color: #333;
      margin: 0;
      padding: 0;
    }

    /* --- LOGIN SECTION --- */
    .login-container {
      max-width: 380px;
      margin: 80px auto;
      padding: 30px;
      background-color: #ffffff;
      border-radius: 12px;
      box-shadow: 0 8px 20px rgba(0, 102, 204, 0.15);
      text-align: center;
      border: 1px solid #b3d9ff;
    }

    .login-container h2 {
      color: #0056b3;
      margin-bottom: 5px;
      font-size: 1.6rem;
    }

    .login-container p {
      color: #666;
      margin-bottom: 20px;
      font-size: 0.9rem;
    }

    .login-container input {
      width: 100%;
      padding: 12px;
      margin: 8px 0;
      border: 1px solid #99ccff;
      border-radius: 6px;
      box-sizing: border-box;
      outline: none;
    }

    .login-container input:focus {
      border-color: #0066cc;
      box-shadow: 0 0 5px rgba(0, 102, 204, 0.3);
    }

    .login-container button {
      width: 100%;
      padding: 12px;
      background-color: #0066cc;
      color: white;
      border: none;
      border-radius: 6px;
      font-weight: bold;
      cursor: pointer;
      margin-top: 10px;
      transition: background 0.3s;
    }

    .login-container button:hover {
      background-color: #004080;
    }

    .error-msg {
      color: #d9534f;
      font-size: 0.85rem;
      margin-top: 10px;
      display: none;
    }

    /* --- APP HEADER & NAVIGATION --- */
    .app-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background-color: #004080;
      color: white;
      padding: 15px 25px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }

    .app-header h1 {
      margin: 0;
      font-size: 1.3rem;
      letter-spacing: 0.5px;
    }

    .user-tag {
      font-size: 0.9rem;
      background-color: #0056b3;
      padding: 3px 8px;
      border-radius: 4px;
      margin-left: 10px;
      font-weight: normal;
    }

    .nav-buttons button {
      background-color: #0066cc;
      color: white;
      border: none;
      padding: 8px 16px;
      margin-left: 8px;
      border-radius: 4px;
      cursor: pointer;
      font-weight: 500;
    }

    .nav-buttons button:hover {
      background-color: #0056b3;
    }

    .btn-logout {
      background-color: #d9534f !important;
    }

    .btn-logout:hover {
      background-color: #c9302c !important;
    }

    /* --- MAIN CONTENT AREA --- */
    main {
      padding: 25px;
      max-width: 1100px;
      margin: 0 auto;
    }

    .tab-content {
      background: white;
      padding: 25px;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
      border: 1px solid #cce5ff;
    }

    h2 {
      color: #004080;
      margin-top: 0;
    }

    /* --- TABLES --- */
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
    }

    table, th, td {
      border: 1px solid #cce5ff;
    }

    th {
      background-color: #cce5ff;
      color: #004080;
      padding: 10px;
      text-align: left;
    }

    td {
      padding: 10px;
    }

    /* --- AUDIT CARDS --- */
    .audit-controls {
      display: flex;
      gap: 15px;
      align-items: center;
      margin-bottom: 20px;
    }

    .audit-summary-cards {
      display: flex;
      gap: 20px;
      margin-bottom: 25px;
    }

    .card {
      flex: 1;
      background-color: #f0f8ff;
      border: 1px solid #b3d9ff;
      border-radius: 8px;
      padding: 15px;
      text-align: center;
    }

    .card h3 {
      margin: 0 0 10px 0;
      font-size: 0.9rem;
      color: #0056b3;
    }

    .card p {
      margin: 0;
      font-size: 1.5rem;
      font-weight: bold;
      color: #004080;
    }

    /* --- MODAL --- */
    .modal {
      position: fixed;
      top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.5);
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .modal-content {
      background: white;
      padding: 25px;
      border-radius: 8px;
      width: 400px;
      border-top: 5px solid #0066cc;
    }
  </style>
</head>
<body>

  <!-- LOGIN CONTAINER -->
  <div id="login-container" class="login-container">
    <h2>RMVillasis Enterprises</h2>
    <p>Mag-login para makapasok sa POS System</p>
    <form onsubmit="handleLogin(event)">
      <input type="text" id="username" placeholder="Username" required>
      <input type="password" id="password" placeholder="Password" required>
      <button type="submit">LOGIN</button>
      <p id="login-error" class="error-msg">Maling Username o Password!</p>
    </form>
  </div>

  <!-- MAIN APP CONTAINER -->
  <div id="app-container" style="display: none;">
    
    <!-- HEADER -->
    <header class="app-header">
      <h1>🏬 RMVillasis Enterprises POS <span class="user-tag" id="current-user-display">User</span></h1>
      <nav class="nav-buttons">
        <button onclick="showTab('pos')">🛒 POS</button>
        <button onclick="showTab('sales')">📝 Sales Management</button>
        <button onclick="showTab('audit')">📊 Monthly Audit</button>
        <button onclick="handleLogout()" class="btn-logout">🚪 Logout</button>
      </nav>
    </header>

    <main>
      <!-- POS SECTION -->
      <section id="pos-section" class="tab-content">
        <h2>🛒 POS Cashier</h2>
        <p>I-click ang button sa ibaba para mag-simula ng sample transaction sa RMVillasis Enterprises.</p>
        <button onclick="addSampleSale()" style="padding: 10px 15px; background: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer;">+ Magdagdag ng Sample Sale (₱150)</button>
      </section>

      <!-- SALES MANAGEMENT SECTION -->
      <section id="sales-section" class="tab-content" style="display: none;">
        <h2>📝 Sales History & Management</h2>
        <table>
          <thead>
            <tr>
              <th>ID</th>
              <th>Petsa</th>
              <th>Mga Item</th>
              <th>Kabuuan</th>
              <th>Aksyon</th>
            </tr>
          </thead>
          <tbody id="sales-history-tbody">
            <!-- Dinamiko ring lalabas dito ang mga sales -->
          </tbody>
        </table>
      </section>

      <!-- MONTHLY AUDIT SECTION -->
      <section id="audit-section" class="tab-content" style="display: none;">
        <h2>📊 RMVillasis Monthly Audit Report</h2>
        
        <div class="audit-controls">
          <label for="audit-month">Pumili ng Buwan:</label>
          <input type="month" id="audit-month" onchange="generateMonthlyAudit()">
          <button onclick="window.print()">🖨️ Print Report</button>
        </div>

        <div class="audit-summary-cards">
          <div class="card">
            <h3>Gross Sales</h3>
            <p id="audit-total-sales">₱0.00</p>
          </div>
          <div class="card">
            <h3>Dami ng Transaksyon</h3>
            <p id="audit-total-orders">0</p>
          </div>
        </div>
      </section>
    </main>

  </div>

  <!-- JAVASCRIPT LOGIC -->
  <script>
    // 1. LOGIN & LOGOUT SYSTEM
    const VALID_USERS = {
      'admin': 'admin123',
      'cashier': '12345'
    };

    function handleLogin(event) {
      event.preventDefault();
      const user = document.getElementById('username').value;
      const pass = document.getElementById('password').value;

      if (VALID_USERS[user] && VALID_USERS[user] === pass) {
        sessionStorage.setItem('isLoggedIn', 'true');
        sessionStorage.setItem('currentUser', user);
        initApp();
      } else {
        document.getElementById('login-error').style.display = 'block';
      }
    }

    function handleLogout() {
      sessionStorage.removeItem('isLoggedIn');
      sessionStorage.removeItem('currentUser');
      location.reload();
    }

    function checkAuthStatus() {
      const isLoggedIn = sessionStorage.getItem('isLoggedIn');
      if (isLoggedIn === 'true') {
        initApp();
      } else {
        document.getElementById('login-container').style.display = 'block';
        document.getElementById('app-container').style.display = 'none';
      }
    }

    function initApp() {
      document.getElementById('login-container').style.display = 'none';
      document.getElementById('app-container').style.display = 'block';
      document.getElementById('current-user-display').innerText = sessionStorage.getItem('currentUser').toUpperCase();
      showTab('pos');
    }

    // 2. TAB NAVIGATION
    function showTab(tabName) {
      document.querySelectorAll('.tab-content').forEach(tab => tab.style.display = 'none');
      document.getElementById(`${tabName}-section`).style.display = 'block';

      if (tabName === 'sales') renderSalesHistory();
      if (tabName === 'audit') {
        const now = new Date();
        document.getElementById('audit-month').value = now.toISOString().slice(0, 7);
        generateMonthlyAudit();
      }
    }

    // 3. SALES MANAGEMENT & AUDIT LOGIC
    function addSampleSale() {
      let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const newSale = {
        id: salesHistory.length + 1,
        date: new Date().toISOString(),
        total: 150,
        items: [{ name: 'Sample Item', price: 150, quantity: 1 }]
      };
      salesHistory.push(newSale);
      localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
      alert('Naidagdag ang sample sale!');
    }

    function renderSalesHistory() {
      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const tbody = document.getElementById('sales-history-tbody');
      tbody.innerHTML = '';

      if(salesHistory.length === 0) {
        tbody.innerHTML = '<tr><td colspan="5">Walang nakatalang benta.</td></tr>';
        return;
      }

      salesHistory.forEach((tx, index) => {
        const itemsSummary = tx.items.map(i => `${i.name} (x${i.quantity})`).join(', ');
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>#${tx.id || index + 1}</td>
          <td>${new Date(tx.date).toLocaleDateString()}</td>
          <td>${itemsSummary}</td>
          <td>₱${tx.total.toFixed(2)}</td>
          <td>
            <button onclick="deleteTransaction(${index})" style="background:#d9534f; color:white; border:none; padding:5px 10px; border-radius:3px; cursor:pointer;">🗑️ Delete</button>
          </td>
        `;
        tbody.appendChild(row);
      });
    }

    function deleteTransaction(index) {
      if (confirm("Sigurado ka bang gusto mong burahin ang transaksyong ito?")) {
        let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
        salesHistory.splice(index, 1);
        localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
        renderSalesHistory();
      }
    }

    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('audit-month').value;
      if (!selectedMonth) return;

      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const filteredSales = salesHistory.filter(sale => {
        return new Date(sale.date).toISOString().slice(0, 7) === selectedMonth;
      });

      let totalSales = filteredSales.reduce((sum, sale) => sum + sale.total, 0);

      document.getElementById('audit-total-sales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('audit-total-orders').innerText = filteredSales.length;
    }

    // Run Auth check on page load
    window.onload = checkAuthStatus;
  </script>
</body>
</html>
