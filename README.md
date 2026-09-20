<!DOCTYPE html>
<html lang="tl">
<head>
  <meta charset="UTF-8">
  <title>RMVillasis Enterprises POS</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

  <!-- LOGIN SECTION -->
  <div id="login-container" class="login-box">
    <h2>RMVillasis Enterprises</h2>
    <p>Secure POS System</p>
    <form id="login-form" onsubmit="handleLogin(event)">
      <input type="text" id="username" placeholder="Username" required>
      <input type="password" id="password" placeholder="Password" required>
      <button type="submit">LOGIN</button>
      <p id="login-error" class="error-msg" style="display:none;">Maling Username o Password!</p>
    </form>
  </div>

  <!-- MAIN APP CONTAINER (Hidden before login) -->
  <div id="app-container" style="display: none;">
    
    <!-- HEADER & NAVIGATION -->
    <header class="app-header">
      <span>User: <strong id="current-user-display">Admin</strong></span>
      <nav>
        <button onclick="showTab('pos')">🛒 POS</button>
        <button onclick="showTab('sales')">📝 Sales Management</button>
        <button onclick="showTab('audit')">📊 Monthly Audit</button>
        <button onclick="handleLogout()" class="btn-logout">🚪 Logout</button>
      </nav>
    </header>

    <!-- POS SECTION -->
    <section id="pos-section" class="tab-content">
      <h2>POS Cashier</h2>
      <!-- Dito ang iyong dating POS Layout / Cart -->
    </section>

    <!-- SALES MANAGEMENT SECTION (EDIT / DELETE / ADD / SUBTRACT) -->
    <section id="sales-section" class="tab-content" style="display: none;">
      <h2>📝 Sales History & Editing</h2>
      <table border="1" class="data-table">
        <thead>
          <tr>
            <th>Transaction ID</th>
            <th>Petsa</th>
            <th>Mga Item</th>
            <th>Kabuuan</th>
            <th>Aksyon</th>
          </tr>
        </thead>
        <tbody id="sales-history-tbody">
          <!-- Dinamiko ring lalabas dito ang mga na-save na benta -->
        </tbody>
      </table>
    </section>

    <!-- MONTHLY AUDIT SECTION -->
    <section id="audit-section" class="tab-content" style="display: none;">
      <h2>📊 Monthly Audit Report</h2>
      <div class="audit-controls">
        <input type="month" id="audit-month" onchange="generateMonthlyAudit()">
        <button onclick="window.print()">🖨️ Print Report</button>
      </div>
      <div class="audit-cards">
        <div>Gross Sales: <strong id="audit-total-sales">₱0.00</strong></div>
        <div>Total Orders: <strong id="audit-total-orders">0</strong></div>
      </div>
    </section>

  </div>

  <!-- MODAL PARA SA PAG-EDIT NG SALES ITEM -->
  <div id="edit-modal" class="modal" style="display:none;">
    <div class="modal-content">
      <h3>Edit Transaction (<span id="edit-tx-id"></span>)</h3>
      <div id="edit-items-container"></div>
      <br>
      <button onclick="saveSalesEdit()" class="btn-save">💾 Save Changes</button>
      <button onclick="closeEditModal()" class="btn-cancel">❌ Cancel</button>
    </div>
  </div>

  <script src="script.js"></script>
</body>
</html>
