<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>RMVillasis Enterprises POS & Inventory</title>
  <style>
    /* --- GENERAL STYLES (Light Blue Theme) --- */
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #e6f2ff;
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
    }

    .user-tag {
      font-size: 0.9rem;
      background-color: #0056b3;
      padding: 3px 8px;
      border-radius: 4px;
      margin-left: 10px;
    }

    .nav-buttons button {
      background-color: #0066cc;
      color: white;
      border: none;
      padding: 8px 16px;
      margin-left: 8px;
      border-radius: 4px;
      cursor: pointer;
    }

    .nav-buttons button:hover {
      background-color: #0056b3;
    }

    .btn-logout {
      background-color: #d9534f !important;
    }

    /* --- MAIN CONTENT AREA --- */
    main {
      padding: 25px;
      max-width: 1150px;
      margin: 0 auto;
    }

    .tab-content {
      background: white;
      padding: 25px;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
      border: 1px solid #cce5ff;
    }

    h2, h3 {
      color: #004080;
      margin-top: 0;
    }

    /* --- FORM STYLES --- */
    .app-form {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 15px;
      background-color: #f0f8ff;
      padding: 20px;
      border-radius: 8px;
      border: 1px solid #b3d9ff;
      margin-bottom: 25px;
    }

    .form-group {
      display: flex;
      flex-direction: column;
    }

    .form-group label {
      font-size: 0.85rem;
      font-weight: bold;
      color: #0056b3;
      margin-bottom: 5px;
    }

    .form-group input, .form-group select {
      padding: 8px;
      border: 1px solid #99ccff;
      border-radius: 4px;
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
      font-size: 0.9rem;
    }

    td {
      padding: 10px;
      font-size: 0.9rem;
    }

    /* --- AUDIT CARDS & BREAKDOWN --- */
    .audit-controls {
      display: flex;
      gap: 15px;
      align-items: center;
      margin-bottom: 20px;
    }

    .audit-summary-cards {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
      gap: 15px;
      margin-bottom: 25px;
    }

    .card {
      background-color: #f0f8ff;
      border: 1px solid #b3d9ff;
      border-radius: 8px;
      padding: 15px;
      text-align: center;
    }

    .card h3 {
      margin: 0 0 10px 0;
      font-size: 0.8rem;
      color: #0056b3;
      text-transform: uppercase;
    }

    .card p {
      margin: 0;
      font-size: 1.25rem;
      font-weight: bold;
      color: #004080;
    }

    .card.expense p { color: #d9534f; }
    .card.profit p { color: #28a745; }

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
      width: 450px;
      border-top: 5px solid #0066cc;
    }
  </style>
</head>
<body>

  <!-- LOGIN CONTAINER -->
  <div id="login-container" class="login-container">
    <h2>RMVillasis Enterprises</h2>
    <p>Please log in to access the POS System</p>
    <form onsubmit="handleLogin(event)">
      <input type="text" id="username" placeholder="Username" required>
      <input type="password" id="password" placeholder="Password" required>
      <button type="submit">LOGIN</button>
      <p id="login-error" class="error-msg">Invalid Username or Password!</p>
    </form>
  </div>

  <!-- MAIN APP CONTAINER -->
  <div id="app-container" style="display: none;">
    
    <!-- HEADER -->
    <header class="app-header">
      <h1>🏬 RMVillasis Enterprises POS <span class="user-tag" id="current-user-display">User</span></h1>
      <nav class="nav-buttons">
        <button onclick="showTab('pos')">🛒 POS Entry</button>
        <button onclick="showTab('inventory')">📦 Inventory</button>
        <button onclick="showTab('sales')">📝 Sales Management</button>
        <button onclick="showTab('audit')">📊 Monthly Audit</button>
        <button onclick="handleLogout()" class="btn-logout">🚪 Logout</button>
      </nav>
    </header>

    <main>
      <!-- POS SECTION -->
      <section id="pos-section" class="tab-content">
        <h2>🛒 Record New Sale</h2>
        <form onsubmit="recordSale(event)" class="app-form">
          <div class="form-group">
            <label>Date of Sale:</label>
            <input type="date" id="pos-date" required>
          </div>
          <div class="form-group">
            <label>Customer Name:</label>
            <input type="text" id="pos-customer" placeholder="e.g., John Doe" required>
          </div>
          <div class="form-group">
            <label>Product Name:</label>
            <input type="text" id="pos-product" placeholder="Item description" required>
          </div>
          <div class="form-group">
            <label>Quantity:</label>
            <input type="number" id="pos-qty" min="1" value="1" required>
          </div>
          <div class="form-group">
            <label>Buying Price / Cost (₱):</label>
            <input type="number" id="pos-buy-price" step="0.01" placeholder="0.00" required>
          </div>
          <div class="form-group">
            <label>Selling Price (₱):</label>
            <input type="number" id="pos-sell-price" step="0.01" placeholder="0.00" required>
          </div>
          <div class="form-group">
            <label>Payment Method:</label>
            <select id="pos-payment-method" required>
              <option value="Cash">Cash</option>
              <option value="GCash">GCash</option>
              <option value="Cheque">Cheque</option>
              <option value="Bank Transfer">Bank Transfer</option>
            </select>
          </div>
          <div class="form-group" style="grid-column: 1 / -1;">
            <button type="submit" style="padding: 10px; background: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">+ Add Sale Transaction</button>
          </div>
        </form>
      </section>

      <!-- INVENTORY SECTION -->
      <section id="inventory-section" class="tab-content" style="display: none;">
        <h2>📦 Product Inventory</h2>
        <form onsubmit="saveInventoryProduct(event)" class="app-form">
          <input type="hidden" id="inv-index">
          <div class="form-group">
            <label>Product Name:</label>
            <input type="text" id="inv-name" placeholder="Item name" required>
          </div>
          <div class="form-group">
            <label>Starting Quantity:</label>
            <input type="number" id="inv-start-qty" min="0" required>
          </div>
          <div class="form-group">
            <label>Default Buying Price (₱):</label>
            <input type="number" id="inv-buy-price" step="0.01" placeholder="0.00" required>
          </div>
          <div class="form-group">
            <label>Default Selling Price (₱):</label>
            <input type="number" id="inv-sell-price" step="0.01" placeholder="0.00" required>
          </div>
          <div class="form-group" style="grid-column: 1 / -1;">
            <button type="submit" id="inv-btn-submit" style="padding: 10px; background: #0066cc; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold;">+ Add Product to Inventory</button>
          </div>
        </form>

        <h3>Current Inventory Levels</h3>
        <table>
          <thead>
            <tr>
              <th>Product Name</th>
              <th>Starting Qty</th>
              <th>Qty Sold</th>
              <th>Remaining / Ending Qty</th>
              <th>Buy Price</th>
              <th>Sell Price</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody id="inventory-tbody"></tbody>
        </table>
      </section>

      <!-- SALES MANAGEMENT SECTION -->
      <section id="sales-section" class="tab-content" style="display: none;">
        <h2>📝 Sales History & Management</h2>
        <table>
          <thead>
            <tr>
              <th>Date</th>
              <th>Customer</th>
              <th>Product (Qty)</th>
              <th>Payment Method</th>
              <th>Total Cost</th>
              <th>Gross Sales</th>
              <th>Gross Profit</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody id="sales-history-tbody"></tbody>
        </table>
      </section>

      <!-- MONTHLY AUDIT SECTION -->
      <section id="audit-section" class="tab-content" style="display: none;">
        <h2>📊 Financial Audit & Payment Breakdown</h2>
        
        <div class="audit-controls">
          <label for="audit-month">Select Month:</label>
          <input type="month" id="audit-month" onchange="generateMonthlyAudit()">
          <button onclick="window.print()">🖨️ Print Report</button>
        </div>

        <!-- SUMMARY CARDS -->
        <h3>Overall Summary</h3>
        <div class="audit-summary-cards">
          <div class="card">
            <h3>Total Gross Sales</h3>
            <p id="audit-total-sales">₱0.00</p>
          </div>
          <div class="card expense">
            <h3>Total Cost</h3>
            <p id="audit-total-cost">₱0.00</p>
          </div>
          <div class="card expense">
            <h3>Expenses</h3>
            <p id="audit-total-expenses">₱0.00</p>
          </div>
          <div class="card profit">
            <h3>Net Profit (Kita)</h3>
            <p id="audit-net-profit">₱0.00</p>
          </div>
        </div>

        <h3>Sales Breakdown by Payment Method</h3>
        <div class="audit-summary-cards">
          <div class="card">
            <h3>Cash Sales</h3>
            <p id="audit-cash-sales">₱0.00</p>
          </div>
          <div class="card">
            <h3>GCash Sales</h3>
            <p id="audit-gcash-sales">₱0.00</p>
          </div>
          <div class="card">
            <h3>Cheque Sales</h3>
            <p id="audit-cheque-sales">₱0.00</p>
          </div>
          <div class="card">
            <h3>Bank Transfer</h3>
            <p id="audit-bank-sales">₱0.00</p>
          </div>
        </div>

        <hr style="border: 0; border-top: 1px solid #cce5ff; margin: 25px 0;">

        <!-- EXPENSE LOGGING SECTION -->
        <h3>💸 Record Monthly Expense / Salary</h3>
        <form style="display: flex; gap: 10px; margin-bottom: 15px;" onsubmit="addExpense(event)">
          <select id="expense-category" required style="padding: 8px;">
            <option value="Salary">Salary Expense</option>
            <option value="Utilities">Utilities (Electricity/Water)</option>
            <option value="Supplies">Supplies & Inventory</option>
            <option value="Other">Other Expenses</option>
          </select>
          <input type="text" id="expense-desc" placeholder="Description / Employee Name" required style="padding: 8px;">
          <input type="number" id="expense-amount" placeholder="Amount (₱)" step="0.01" required style="padding: 8px;">
          <button type="submit" style="background:#0066cc; color:white; border:none; border-radius:4px; padding:8px 15px; cursor:pointer;">+ Add Expense</button>
        </form>

        <!-- EXPENSE TABLE -->
        <h3>Expense Log</h3>
        <table>
          <thead>
            <tr>
              <th>Category</th>
              <th>Description</th>
              <th>Amount</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody id="expense-tbody"></tbody>
        </table>
      </section>
    </main>

  </div>

  <!-- EDIT SALE MODAL -->
  <div id="edit-modal" class="modal" style="display:none;">
    <div class="modal-content">
      <h3>Edit Sale Details</h3>
      <form onsubmit="saveSaleEdit(event)">
        <input type="hidden" id="edit-index">
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Date of Sale:</label>
          <input type="date" id="edit-date" required>
        </div>
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Customer Name:</label>
          <input type="text" id="edit-customer" required>
        </div>
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Product Name:</label>
          <input type="text" id="edit-product" required>
        </div>
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Quantity:</label>
          <input type="number" id="edit-qty" min="1" required>
        </div>
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Buying Price (Cost):</label>
          <input type="number" id="edit-buy-price" step="0.01" required>
        </div>
        <div class="form-group" style="margin-bottom: 10px;">
          <label>Selling Price:</label>
          <input type="number" id="edit-sell-price" step="0.01" required>
        </div>
        <div class="form-group" style="margin-bottom: 15px;">
          <label>Payment Method:</label>
          <select id="edit-payment-method" required>
            <option value="Cash">Cash</option>
            <option value="GCash">GCash</option>
            <option value="Cheque">Cheque</option>
            <option value="Bank Transfer">Bank Transfer</option>
          </select>
        </div>
        <button type="submit" style="background:#0066cc; color:white; padding:8px 12px; border:none; border-radius:4px; cursor:pointer;">💾 Save Changes</button>
        <button type="button" onclick="closeEditModal()" style="background:#6c757d; color:white; padding:8px 12px; border:none; border-radius:4px; cursor:pointer;">❌ Cancel</button>
      </form>
    </div>
  </div>

  <!-- JAVASCRIPT LOGIC -->
  <script>
    // 1. AUTHENTICATION SYSTEM
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
      if (sessionStorage.getItem('isLoggedIn') === 'true') {
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
      
      const today = new Date().toISOString().split('T')[0];
      document.getElementById('pos-date').value = today;

      showTab('pos');
    }

    // 2. NAVIGATION
    function showTab(tabName) {
      document.querySelectorAll('.tab-content').forEach(tab => tab.style.display = 'none');
      document.getElementById(`${tabName}-section`).style.display = 'block';

      if (tabName === 'inventory') renderInventory();
      if (tabName === 'sales') renderSalesHistory();
      if (tabName === 'audit') {
        const now = new Date();
        document.getElementById('audit-month').value = now.toISOString().slice(0, 7);
        generateMonthlyAudit();
      }
    }

    // 3. INVENTORY MANAGEMENT LOGIC
    function saveInventoryProduct(e) {
      e.preventDefault();
      const index = document.getElementById('inv-index').value;
      const name = document.getElementById('inv-name').value.trim();
      const startingQty = parseInt(document.getElementById('inv-start-qty').value);
      const buyPrice = parseFloat(document.getElementById('inv-buy-price').value);
      const sellPrice = parseFloat(document.getElementById('inv-sell-price').value);

      let inventory = JSON.parse(localStorage.getItem('inventoryList')) || [];

      if (index !== "") {
        // Edit existing product
        inventory[index] = { name, startingQty, buyPrice, sellPrice };
      } else {
        // Add new product
        inventory.push({ name, startingQty, buyPrice, sellPrice });
      }

      localStorage.setItem('inventoryList', JSON.stringify(inventory));
      alert('Product saved to Inventory!');

      e.target.reset();
      document.getElementById('inv-index').value = "";
      document.getElementById('inv-btn-submit').innerText = "+ Add Product to Inventory";
      renderInventory();
    }

    function renderInventory() {
      const inventory = JSON.parse(localStorage.getItem('inventoryList')) || [];
      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const tbody = document.getElementById('inventory-tbody');
      tbody.innerHTML = '';

      if (inventory.length === 0) {
        tbody.innerHTML = '<tr><td colspan="7">No products in inventory yet.</td></tr>';
        return;
      }

      inventory.forEach((item, idx) => {
        // Compute total quantity sold for this product
        const qtySold = salesHistory
          .filter(s => s.product.toLowerCase() === item.name.toLowerCase())
          .reduce((sum, s) => sum + s.qty, 0);

        const remainingQty = item.startingQty - qtySold;

        const row = document.createElement('tr');
        row.innerHTML = `
          <td><b>${item.name}</b></td>
          <td>${item.startingQty}</td>
          <td>${qtySold}</td>
          <td style="color: ${remainingQty <= 5 ? 'red' : 'green'}; font-weight: bold;">${remainingQty}</td>
          <td>₱${item.buyPrice.toFixed(2)}</td>
          <td>₱${item.sellPrice.toFixed(2)}</td>
          <td>
            <button onclick="editInventoryProduct(${idx})" style="background:#0066cc; color:white; border:none; padding:4px 8px; border-radius:3px; cursor:pointer;">✏️ Edit</button>
            <button onclick="deleteInventoryProduct(${idx})" style="background:#d9534f; color:white; border:none; padding:4px 8px; border-radius:3px; cursor:pointer;">🗑️ Delete</button>
          </td>
        `;
        tbody.appendChild(row);
      });
    }

    function editInventoryProduct(idx) {
      const inventory = JSON.parse(localStorage.getItem('inventoryList')) || [];
      const item = inventory[idx];

      document.getElementById('inv-index').value = idx;
      document.getElementById('inv-name').value = item.name;
      document.getElementById('inv-start-qty').value = item.startingQty;
      document.getElementById('inv-buy-price').value = item.buyPrice;
      document.getElementById('inv-sell-price').value = item.sellPrice;
      document.getElementById('inv-btn-submit').innerText = "💾 Save Inventory Changes";
    }

    function deleteInventoryProduct(idx) {
      if (confirm("Are you sure you want to delete this product from inventory?")) {
        let inventory = JSON.parse(localStorage.getItem('inventoryList')) || [];
        inventory.splice(idx, 1);
        localStorage.setItem('inventoryList', JSON.stringify(inventory));
        renderInventory();
      }
    }

    // 4. POS & SALES RECORDING
    function recordSale(e) {
      e.preventDefault();
      const saleDate = document.getElementById('pos-date').value;
      const customer = document.getElementById('pos-customer').value;
      const product = document.getElementById('pos-product').value;
      const qty = parseInt(document.getElementById('pos-qty').value);
      const buyPrice = parseFloat(document.getElementById('pos-buy-price').value);
      const sellPrice = parseFloat(document.getElementById('pos-sell-price').value);
      const paymentMethod = document.getElementById('pos-payment-method').value;

      const totalCost = buyPrice * qty;
      const grossSales = sellPrice * qty;
      const profit = grossSales - totalCost;

      const newSale = {
        date: saleDate,
        customer,
        product,
        qty,
        buyPrice,
        sellPrice,
        paymentMethod,
        totalCost,
        grossSales,
        profit
      };

      let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      salesHistory.push(newSale);
      localStorage.setItem('salesHistory', JSON.stringify(salesHistory));

      alert('Sale recorded successfully!');
      e.target.reset();
      
      const today = new Date().toISOString().split('T')[0];
      document.getElementById('pos-date').value = today;
      document.getElementById('pos-qty').value = 1;
    }

    function renderSalesHistory() {
      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const tbody = document.getElementById('sales-history-tbody');
      tbody.innerHTML = '';

      if (salesHistory.length === 0) {
        tbody.innerHTML = '<tr><td colspan="8">No sales recorded yet.</td></tr>';
        return;
      }

      salesHistory.forEach((sale, idx) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${sale.date}</td>
          <td>${sale.customer}</td>
          <td>${sale.product} (x${sale.qty})</td>
          <td><b>${sale.paymentMethod || 'Cash'}</b></td>
          <td>₱${sale.totalCost.toFixed(2)}</td>
          <td>₱${sale.grossSales.toFixed(2)}</td>
          <td style="color: green; font-weight: bold;">₱${sale.profit.toFixed(2)}</td>
          <td>
            <button onclick="openEditModal(${idx})" style="background:#0066cc; color:white; border:none; padding:4px 8px; border-radius:3px; cursor:pointer;">✏️ Edit</button>
            <button onclick="deleteSale(${idx})" style="background:#d9534f; color:white; border:none; padding:4px 8px; border-radius:3px; cursor:pointer;">🗑️ Delete</button>
          </td>
        `;
        tbody.appendChild(row);
      });
    }

    // 5. EDIT & DELETE SALES
    function openEditModal(idx) {
      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const sale = salesHistory[idx];

      document.getElementById('edit-index').value = idx;
      document.getElementById('edit-date').value = sale.date;
      document.getElementById('edit-customer').value = sale.customer;
      document.getElementById('edit-product').value = sale.product;
      document.getElementById('edit-qty').value = sale.qty;
      document.getElementById('edit-buy-price').value = sale.buyPrice;
      document.getElementById('edit-sell-price').value = sale.sellPrice;
      document.getElementById('edit-payment-method').value = sale.paymentMethod || 'Cash';

      document.getElementById('edit-modal').style.display = 'flex';
    }

    function closeEditModal() {
      document.getElementById('edit-modal').style.display = 'none';
    }

    function saveSaleEdit(e) {
      e.preventDefault();
      const idx = document.getElementById('edit-index').value;
      let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];

      const qty = parseInt(document.getElementById('edit-qty').value);
      const buyPrice = parseFloat(document.getElementById('edit-buy-price').value);
      const sellPrice = parseFloat(document.getElementById('edit-sell-price').value);

      salesHistory[idx].date = document.getElementById('edit-date').value;
      salesHistory[idx].customer = document.getElementById('edit-customer').value;
      salesHistory[idx].product = document.getElementById('edit-product').value;
      salesHistory[idx].qty = qty;
      salesHistory[idx].buyPrice = buyPrice;
      salesHistory[idx].sellPrice = sellPrice;
      salesHistory[idx].paymentMethod = document.getElementById('edit-payment-method').value;
      salesHistory[idx].totalCost = buyPrice * qty;
      salesHistory[idx].grossSales = sellPrice * qty;
      salesHistory[idx].profit = (sellPrice * qty) - (buyPrice * qty);

      localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
      closeEditModal();
      renderSalesHistory();
    }

    function deleteSale(idx) {
      if (confirm("Are you sure you want to delete this sale record?")) {
        let salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
        salesHistory.splice(idx, 1);
        localStorage.setItem('salesHistory', JSON.stringify(salesHistory));
        renderSalesHistory();
      }
    }

    // 6. EXPENSES & MONTHLY AUDIT LOGIC
    function addExpense(e) {
      e.preventDefault();
      const category = document.getElementById('expense-category').value;
      const desc = document.getElementById('expense-desc').value;
      const amount = parseFloat(document.getElementById('expense-amount').value);
      const month = document.getElementById('audit-month').value;

      let expenses = JSON.parse(localStorage.getItem('expensesHistory')) || [];
      expenses.push({ category, desc, amount, month, date: new Date().toISOString() });
      localStorage.setItem('expensesHistory', JSON.stringify(expenses));

      document.getElementById('expense-desc').value = '';
      document.getElementById('expense-amount').value = '';
      generateMonthlyAudit();
    }

    function deleteExpense(index) {
      let expenses = JSON.parse(localStorage.getItem('expensesHistory')) || [];
      expenses.splice(index, 1);
      localStorage.setItem('expensesHistory', JSON.stringify(expenses));
      generateMonthlyAudit();
    }

    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('audit-month').value;
      if (!selectedMonth) return;

      const salesHistory = JSON.parse(localStorage.getItem('salesHistory')) || [];
      const filteredSales = salesHistory.filter(s => s.date.slice(0, 7) === selectedMonth);
      
      const totalSales = filteredSales.reduce((sum, s) => sum + s.grossSales, 0);
      const totalProductCost = filteredSales.reduce((sum, s) => sum + s.totalCost, 0);

      // Separate payments calculation
      let cashTotal = 0;
      let gcashTotal = 0;
      let chequeTotal = 0;
      let bankTotal = 0;

      filteredSales.forEach(s => {
        const method = s.paymentMethod || 'Cash';
        if (method === 'Cash') cashTotal += s.grossSales;
        else if (method === 'GCash') gcashTotal += s.grossSales;
        else if (method === 'Cheque') chequeTotal += s.grossSales;
        else if (method === 'Bank Transfer') bankTotal += s.grossSales;
      });

      const expensesHistory = JSON.parse(localStorage.getItem('expensesHistory')) || [];
      const filteredExpenses = expensesHistory.filter(e => e.month === selectedMonth);
      const totalExpenses = filteredExpenses.reduce((sum, e) => sum + e.amount, 0);

      const tbody = document.getElementById('expense-tbody');
      tbody.innerHTML = '';

      filteredExpenses.forEach((exp, idx) => {
        tbody.innerHTML += `
          <tr>
            <td>${exp.category}</td>
            <td>${exp.desc}</td>
            <td>₱${exp.amount.toFixed(2)}</td>
            <td><button onclick="deleteExpense(${idx})" style="background:#d9534f; color:white; border:none; padding:4px 8px; border-radius:3px; cursor:pointer;">Delete</button></td>
          </tr>
        `;
      });

      if (filteredExpenses.length === 0) {
        tbody.innerHTML = '<tr><td colspan="4">No expenses logged for this month.</td></tr>';
      }

      // Compute Net Profit
      const netProfit = totalSales - totalProductCost - totalExpenses;

      document.getElementById('audit-total-sales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('audit-total-cost').innerText = `₱${totalProductCost.toFixed(2)}`;
      document.getElementById('audit-total-expenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('audit-net-profit').innerText = `₱${netProfit.toFixed(2)}`;

      document.getElementById('audit-cash-sales').innerText = `₱${cashTotal.toFixed(2)}`;
      document.getElementById('audit-gcash-sales').innerText = `₱${gcashTotal.toFixed(2)}`;
      document.getElementById('audit-cheque-sales').innerText = `₱${chequeTotal.toFixed(2)}`;
      document.getElementById('audit-bank-sales').innerText = `₱${bankTotal.toFixed(2)}`;
    }

    // Run Auth check on page load
    window.onload = checkAuthStatus;
  </script>
</body>
</html>
