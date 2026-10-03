<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>RMVillasis Enterprises POS, Daily Report & Audit</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    body { background-color: #f4f6f9; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    .navbar { background-color: #0d47a1; }
    .card { border-radius: 10px; border: none; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
    .nav-pills .nav-link.active { background-color: #1976d2; }
    .nav-pills .nav-link { color: #fff; margin-right: 5px; }
    .credit-fields, .container-fields, .multi-payment-fields { display: none; background-color: #f8f9fa; border-radius: 8px; padding: 15px; margin-top: 15px; border: 1px dashed #cbd5e1; }
    .col-action { width: 45px; text-align: center; vertical-align: middle; }
    .stat-card { border-left: 4px solid #1976d2; }
    #loginOverlay {
      position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
      background: rgba(13, 71, 161, 0.85); z-index: 9999;
      display: flex; justify-content: center; align-items: center;
    }
    @media print {
      body { background-color: #fff !important; color: #000 !important; font-size: 12pt; }
      .navbar, #loginOverlay, .btn, .nav, .modal, .no-print { display: none !important; }
      .card { border: none !important; box-shadow: none !important; padding: 0 !important; }
      .container { max-width: 100% !important; padding: 0 !important; margin: 0 !important; }
      .tab-pane { display: block !important; opacity: 1 !important; }
    }
  </style>
</head>
<body>

  <!-- LOGIN OVERLAY -->
  <div id="loginOverlay">
    <div class="card p-4 shadow-lg" style="width: 380px; border-top: 5px solid #1976d2;">
      <div class="text-center mb-3">
        <i class="fa-solid fa-store fa-3x text-primary mb-2"></i>
        <h4 class="fw-bold">RMVillasis Enterprises</h4>
        <p class="text-muted small">Mangyaring mag-log in upang magpatuloy</p>
      </div>
      <form id="loginForm">
        <div class="mb-3">
          <label class="form-label fw-semibold">Username:</label>
          <input type="text" id="loginUsername" class="form-control" placeholder="admin" required>
        </div>
        <div class="mb-3">
          <label class="form-label fw-semibold">Password:</label>
          <input type="password" id="loginPassword" class="form-control" placeholder="••••••••" required>
        </div>
        <div id="loginError" class="alert alert-danger p-2 small d-none">Mali ang username o password!</div>
        <button type="submit" class="btn btn-primary w-100 fw-bold py-2"><i class="fa-solid fa-right-to-bracket me-2"></i>Log In</button>
      </form>
    </div>
  </div>

  <!-- Navbar -->
  <nav class="navbar navbar-dark expand-lg mb-4">
    <div class="container-fluid">
      <a class="navbar-brand fw-bold fs-4" href="#"><i class="fa-solid fa-store me-2"></i>RMVillasis Enterprises</a>
      <ul class="nav nav-pills me-auto" id="mainTabs" role="tablist">
        <li class="nav-item"><button class="nav-link active" id="pos-tab" data-bs-toggle="pill" data-bs-target="#pos-content" type="button"><i class="fa-solid fa-cart-shopping me-1"></i> POS Entry</button></li>
        <li class="nav-item"><button class="nav-link" id="daily-tab" data-bs-toggle="pill" data-bs-target="#daily-content" type="button" onclick="generateDailyReport()"><i class="fa-solid fa-calendar-day me-1"></i> Daily Report</button></li>
        <li class="nav-item"><button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button" onclick="renderCreditTable(); renderStandalonePayments();"><i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments</button></li>
        <li class="nav-item"><button class="nav-link" id="search-tab" data-bs-toggle="pill" data-bs-target="#search-content" type="button"><i class="fa-solid fa-magnifying-glass me-1"></i> Order Lookup & Last Price</button></li>
        <li class="nav-item"><button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTables();"><i class="fa-solid fa-boxes-stacked me-1"></i> Inventory</button></li>
        <li class="nav-item"><button class="nav-link" id="expenses-tab" data-bs-toggle="pill" data-bs-target="#expenses-content" type="button" onclick="renderStandaloneExpensesLedger()"><i class="fa-solid fa-receipt me-1"></i> Salary & Expenses</button></li>
        <li class="nav-item"><button class="nav-link" id="boss-tab" data-bs-toggle="pill" data-bs-target="#boss-content" type="button" onclick="renderStandaloneBossLedger()"><i class="fa-solid fa-user-tie me-1"></i> D/Eco Boss</button></li>
        <li class="nav-item admin-only"><button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()"><i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit</button></li>
      </ul>
      <div class="d-flex align-items-center gap-2">
        <button class="btn btn-success btn-sm fw-semibold" onclick="manualSaveData()"><i class="fa-solid fa-floppy-disk me-1"></i> Save Data</button>
        <button class="btn btn-outline-light btn-sm fw-semibold" onclick="location.reload()"><i class="fa-solid fa-rotate me-1"></i> Refresh</button>
        <span class="text-white fw-bold ms-2" id="currentUserName">User</span>
      </div>
    </div>
  </nav>

  <div class="container pb-5">
    <div class="tab-content" id="mainTabsContent">

      <!-- 1. POS ENTRY TAB -->
      <div class="tab-pane fade show active" id="pos-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-cash-register me-2"></i>Record New Transaction</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()"><i class="fa-solid fa-print me-1"></i> I-print</button>
          </div>
         
          <div class="alert alert-info py-2 mb-3 d-flex justify-content-between align-items-center">
            <div><i class="fa-solid fa-info-circle me-1"></i> <strong>Mode:</strong> Inventory Only Mode (Walang presyo).</div>
            <div class="form-check form-switch mb-0">
              <input class="form-check-input" type="checkbox" id="inventoryOnlyMode" onchange="toggleInventoryOnlyMode()">
              <label class="form-check-label fw-semibold" for="inventoryOnlyMode">Inventory Only Mode</label>
            </div>
          </div>

          <form id="posForm">
            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Date of Sale / Log:</label>
                <input type="date" id="saleDate" class="form-control" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer Name:</label>
                <input type="text" id="customerName" class="form-control" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Location / Uri:</label>
                <select id="transactionLocation" class="form-select" required>
                  <option value="Hiway">Hiway</option>
                  <option value="Byahe">Byahe</option>
                  <option value="Inventory Only">Inventory Only</option>
                </select>
              </div>
            </div>

            <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-cart-plus me-2"></i>Items / Products</h6>
            <div class="table-responsive mb-3">
              <table class="table table-bordered align-middle" id="posItemsTable">
                <thead class="table-light">
                  <tr>
                    <th>Product Name</th>
                    <th>Description</th>
                    <th style="width: 100px;">Qty</th>
                    <th style="width: 130px;" class="price-col">Cost (₱)</th>
                    <th style="width: 130px;" class="price-col">Price (₱)</th>
                    <th style="width: 130px;" class="price-col">Subtotal (₱)</th>
                    <th class="col-action"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="posItemsBody"></tbody>
              </table>
              <button type="button" class="btn btn-sm btn-outline-primary" onclick="addPosRow()"><i class="fa-solid fa-plus me-1"></i> Add Another Item</button>
            </div>

            <div class="card p-3 bg-light border mb-3" id="containerSectionBox">
              <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-box-open me-2"></i>Container Details</h6>
              <div class="row g-3">
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Container Status:</label>
                  <select id="containerStatus" class="form-select" onchange="toggleContainerFields()">
                    <option value="NONE">Walang Container</option>
                    <option value="HIRAM">Hiram</option>
                    <option value="DEPOSIT">May Deposito</option>
                  </select>
                </div>
                <div class="col-md-4 container-qty-group" style="display: none;">
                  <label class="form-label fw-semibold">Ilang Pcs:</label>
                  <input type="number" step="any" min="0.5" id="containerQty" class="form-control" value="1">
                </div>
                <div class="col-md-4 container-deposit-group" style="display: none;">
                  <label class="form-label fw-semibold">Deposit Rate (₱):</label>
                  <input type="number" step="0.01" min="0" id="containerDepositRate" class="form-control" placeholder="0.00" oninput="calculateTotal()">
                </div>
              </div>
            </div>

            <div class="row g-3 mt-2" id="financialSectionBox">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="number" step="0.01" id="totalAmount" class="form-control bg-light fs-5 fw-bold text-primary" readonly placeholder="0.00">
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Transaction Type:</label>
                <select id="paymentType" class="form-select" onchange="toggleCreditFields()">
                  <option value="FULL">Paid in Full</option>
                  <option value="PARTIAL">Partial Payment</option>
                  <option value="CREDIT">Full Credit</option>
                </select>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Payment Method:</label>
                <select id="paymentMethod" class="form-select" onchange="togglePaymentMethodFields()">
                  <option value="Cash">Cash</option>
                  <option value="Byahe Cash">Byahe Cash</option>
                  <option value="GCash">GCash</option>
                  <option value="Bank Transfer">Bank Transfer (BT)</option>
                  <option value="Cheque">Cheque</option>
                  <option value="Multi-Payment">Multi-Payment (Cash + GCash/Iba pa)</option>
                </select>
              </div>
            </div>

            <!-- MULTI-PAYMENT BREAKDOWN -->
            <div id="multiPaymentFields" class="multi-payment-fields">
              <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-wallet me-2"></i>Multi-Payment Breakdown</h6>
              <div class="row g-2">
                <div class="col-md-3"><label class="form-label small fw-semibold">Cash (₱):</label><input type="number" step="0.01" id="multiCashAmt" class="form-control form-control-sm" value="0.00"></div>
                <div class="col-md-3"><label class="form-label small fw-semibold">GCash (₱):</label><input type="number" step="0.01" id="multiGcashAmt" class="form-control form-control-sm" value="0.00"></div>
                <div class="col-md-3"><label class="form-label small fw-semibold">Bank Transfer (₱):</label><input type="number" step="0.01" id="multiBTAmt" class="form-control form-control-sm" value="0.00"></div>
                <div class="col-md-3"><label class="form-label small fw-semibold">Byahe Cash (₱):</label><input type="number" step="0.01" id="multiOtherAmt" class="form-control form-control-sm" value="0.00"></div>
              </div>
            </div>

            <div id="creditFieldsSection" class="credit-fields">
              <h6 class="text-secondary fw-bold mb-3"><i class="fa-solid fa-file-invoice-dollar me-2"></i>Utang Details</h6>
              <div class="row g-3">
                <div class="col-md-4"><label class="form-label fw-semibold">Amount Paid Now (₱):</label><input type="number" step="0.01" id="amountPaidNow" class="form-control" value="0.00" oninput="calculateBalance()"></div>
                <div class="col-md-4"><label class="form-label fw-semibold">Remaining Balance (₱):</label><input type="number" step="0.01" id="remainingBalance" class="form-control bg-light" readonly value="0.00"></div>
                <div class="col-md-4"><label class="form-label fw-semibold">Due Date:</label><input type="date" id="dueDate" class="form-control"></div>
              </div>
            </div>

            <div class="mt-4 text-end">
              <button type="submit" class="btn btn-success btn-lg px-4"><i class="fa-solid fa-check me-2"></i>Save Transaction</button>
            </div>
          </form>
        </div>
      </div>

      <!-- 2. DAILY REPORT TAB -->
      <div class="tab-pane fade" id="daily-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-calendar-day me-2"></i>Daily Sales & Encoded Logs</h4>
            <div class="d-flex gap-2 align-items-center">
              <label class="fw-bold me-1">Select Date:</label>
              <input type="date" id="dailyReportDate" class="form-control" onchange="generateDailyReport()">
              <button class="btn btn-outline-primary ms-2" onclick="window.print()"><i class="fa-solid fa-print me-1"></i> Print</button>
            </div>
          </div>

          <div class="row g-3 mb-4">
            <div class="col-md-3"><div class="card p-3 stat-card bg-light"><span class="text-muted small fw-bold">HIWAY SALES / PROFIT</span><div class="mt-1"><span class="text-primary fw-bold" id="dailyHiwaySales">₱0.00</span><br><span class="text-success small fw-bold" id="dailyHiwayProfit">₱0.00</span></div></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;"><span class="text-muted small fw-bold">BYAHE SALES / PROFIT</span><div class="mt-1"><span class="text-primary fw-bold" id="dailyByaheSales">₱0.00</span><br><span class="text-success small fw-bold" id="dailyByaheProfit">₱0.00</span></div></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;"><span class="text-muted small fw-bold">TOTAL COLLECTION & NET</span><div class="mt-1"><span class="text-success fw-bold" id="dailyTotalCollected">₱0.00</span><br><span class="text-success fw-bold" id="dailyTotalNetProfit">₱0.00</span></div></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;"><span class="text-muted small fw-bold">TOTAL SALES & TX</span><div class="mt-1"><span class="text-primary fw-bold" id="dailyTotalSales">₱0.00</span><br><span class="text-warning fw-bold" id="dailyTxCount">0</span></div></div></div>
          </div>

          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr><th>#</th><th>Customer</th><th>Location</th><th>Products</th><th>Container</th><th>Cost</th><th>Total</th><th>Paid</th><th>Balance</th><th>Net Profit</th><th>Payment Method</th><th class="no-print">Action</th></tr>
              </thead>
              <tbody id="dailyTableBody"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 3. UTANG & PAYMENTS TAB -->
      <div class="tab-pane fade" id="credit-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-users-viewfinder me-2"></i>Utang & Payments</h4>
          <div class="table-responsive">
            <table class="table table-hover align-middle">
              <thead class="table-dark"><tr><th>Customer</th><th>Location</th><th>Product</th><th>Cost</th><th>Paid</th><th>Balance</th><th>Due Date</th><th>Status</th><th class="no-print">Action</th></tr></thead>
              <tbody id="creditTableBody"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 3.5 ORDER LOOKUP -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-magnifying-glass me-2"></i>Customer Order Lookup</h4>
          <div class="row g-2 mb-4">
            <div class="col-md-9"><input type="text" id="searchCustomerInput" class="form-control" placeholder="Pangalan ng Customer..."></div>
            <div class="col-md-3"><button class="btn btn-primary w-100 fw-bold" onclick="searchCustomerOrder()">Search</button></div>
          </div>
          <div id="searchResultContainer" style="display: none;">
            <div class="card bg-light border-primary p-3 mb-4">
              <h5 class="text-primary fw-bold">Huling Order ni <span id="lastOrderCustomer">-</span></h5>
              <p class="mb-1">Petsa: <span id="lastOrderDate">-</span></p>
            </div>
          </div>
          <div id="noCustomerFound" class="alert alert-warning text-center d-none">Walang nahanap na record.</div>
        </div>
      </div>

      <!-- 4. INVENTORY TAB -->
      <div class="tab-pane fade" id="inventory-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory</h4>
          <div class="table-responsive"><table class="table table-bordered"><tbody id="inventoryTableBody"></tbody></table></div>
        </div>
      </div>

      <!-- 5. EXPENSES TAB -->
      <div class="tab-pane fade" id="expenses-content">
        <div class="card p-4"><h4 class="card-title text-primary mb-4"><i class="fa-solid fa-receipt me-2"></i>Salary & Expenses</h4></div>
      </div>

      <!-- 6. BOSS TAB -->
      <div class="tab-pane fade" id="boss-content">
        <div class="card p-4"><h4 class="card-title text-primary mb-4"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Ledger</h4></div>
      </div>

      <!-- 7. AUDIT TAB -->
      <div class="tab-pane fade" id="audit-content">
        <div class="card p-4"><h4 class="card-title text-primary mb-4"><i class="fa-solid fa-chart-pie me-2"></i>Monthly Audit</h4></div>
      </div>

    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    let users = [{ id: 1, name: "Admin", username: "admin", password: "password", role: "Admin" }];
    let currentUser = JSON.parse(localStorage.getItem('rmv_current_user')) || null;
    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let inventory = JSON.parse(localStorage.getItem('rmv_inventory')) || [];

    function getTodayDateString() {
      const now = new Date();
      return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;
    }

    let todayFormatted = getTodayDateString();
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;

    window.onload = function() {
      addPosRow();
      if (currentUser) {
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('currentUserName').innerText = `${currentUser.name} (${currentUser.role})`;
      }
      generateDailyReport();
      renderCreditTable();
    };

    function saveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_inventory', JSON.stringify(inventory));
    }

    function manualSaveData() {
      saveData();
      alert('Data saved successfully!');
    }

    document.getElementById('loginForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const u = document.getElementById('loginUsername').value.trim();
      const p = document.getElementById('loginPassword').value.trim();
      const found = users.find(user => user.username === u && user.password === p);
      if (found) {
        currentUser = found;
        localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('currentUserName').innerText = `${currentUser.name} (${currentUser.role})`;
      } else {
        document.getElementById('loginError').classList.remove('d-none');
      }
    });

    function toggleInventoryOnlyMode() {
      const isInv = document.getElementById('inventoryOnlyMode').checked;
      document.querySelectorAll('.price-col').forEach(c => c.style.display = isInv ? 'none' : '');
      document.getElementById('containerSectionBox').style.display = isInv ? 'none' : 'block';
      document.getElementById('financialSectionBox').style.display = isInv ? 'none' : 'flex';
      document.getElementById('transactionLocation').value = isInv ? 'Inventory Only' : 'Hiway';
    }

    function togglePaymentMethodFields() {
      const method = document.getElementById('paymentMethod').value;
      document.getElementById('multiPaymentFields').style.display = method === 'Multi-Payment' ? 'block' : 'none';
    }

    function addPosRow() {
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now() + Math.random().toString(36).substring(2, 5);
      tbody.insertAdjacentHTML('beforeend', `
        <tr id="row-${rowId}">
          <td><input type="text" class="form-control form-control-sm pos-name" placeholder="Pangalan" required></td>
          <td><input type="text" class="form-control form-control-sm pos-desc" placeholder="Desc"></td>
          <td><input type="number" step="any" min="0.5" class="form-control form-control-sm pos-qty" value="1" oninput="calculateTotal()" required></td>
          <td class="price-col"><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td class="price-col"><input type="number" step="0.01" class="form-control form-control-sm pos-price" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td class="price-col"><input type="number" step="0.01" class="form-control form-control-sm bg-light pos-subtotal" readonly></td>
          <td class="col-action"><button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="this.closest('tr').remove(); calculateTotal();"><i class="fa-solid fa-trash-can"></i></button></td>
        </tr>
      `);
      calculateTotal();
    }

    function toggleContainerFields() {
      const status = document.getElementById('containerStatus').value;
      document.querySelector('.container-qty-group').style.display = status !== 'NONE' ? 'block' : 'none';
      document.querySelector('.container-deposit-group').style.display = status === 'DEPOSIT' ? 'block' : 'none';
      calculateTotal();
    }

    function calculateTotal() {
      if (document.getElementById('inventoryOnlyMode').checked) return;
      let total = 0;
      document.querySelectorAll('#posItemsBody tr').forEach(row => {
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const price = parseFloat(row.querySelector('.pos-price').value) || 0;
        const sub = qty * price;
        row.querySelector('.pos-subtotal').value = sub.toFixed(2);
        total += sub;
      });
      if (document.getElementById('containerStatus').value === 'DEPOSIT') {
        total += (parseFloat(document.getElementById('containerQty').value) || 0) * (parseFloat(document.getElementById('containerDepositRate').value) || 0);
      }
      document.getElementById('totalAmount').value = total.toFixed(2);
      calculateBalance();
    }

    function toggleCreditFields() {
      const type = document.getElementById('paymentType').value;
      document.getElementById('creditFieldsSection').style.display = type !== 'FULL' ? 'block' : 'none';
      document.getElementById('amountPaidNow').value = type === 'FULL' ? document.getElementById('totalAmount').value : '0.00';
      calculateBalance();
    }

    function calculateBalance() {
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paid = document.getElementById('paymentType').value === 'FULL' ? total : (parseFloat(document.getElementById('amountPaidNow').value) || 0);
      document.getElementById('remainingBalance').value = Math.max(0, total - paid).toFixed(2);
    }

    document.getElementById('posForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const isInv = document.getElementById('inventoryOnlyMode').checked;
      const date = document.getElementById('saleDate').value;
      const cust = document.getElementById('customerName').value.trim();
      const loc = document.getElementById('transactionLocation').value;

      let total = 0, paid = 0, balance = 0, status = "PAID", method = "Cash", payments = [];
      if (!isInv) {
        total = parseFloat(document.getElementById('totalAmount').value) || 0;
        method = document.getElementById('paymentMethod').value;
        const pType = document.getElementById('paymentType').value;
        paid = pType === 'FULL' ? total : (parseFloat(document.getElementById('amountPaidNow').value) || 0);
        balance = Math.max(0, total - paid);
        status = balance > 0 && paid > 0 ? "PARTIAL" : (balance > 0 ? "UNPAID" : "PAID");

        if (method === 'Multi-Payment') {
          if (parseFloat(document.getElementById('multiCashAmt').value) > 0) payments.push({ amount: parseFloat(document.getElementById('multiCashAmt').value), method: 'Cash', date });
          if (parseFloat(document.getElementById('multiGcashAmt').value) > 0) payments.push({ amount: parseFloat(document.getElementById('multiGcashAmt').value), method: 'GCash', date });
          if (parseFloat(document.getElementById('multiBTAmt').value) > 0) payments.push({ amount: parseFloat(document.getElementById('multiBTAmt').value), method: 'Bank Transfer', date });
          if (parseFloat(document.getElementById('multiOtherAmt').value) > 0) payments.push({ amount: parseFloat(document.getElementById('multiOtherAmt').value), method: 'Byahe Cash', date });
        } else if (paid > 0) {
          payments.push({ amount: paid, method, date });
        }
      }

      let itemsList = [], productSummary = [], totalCost = 0;
      document.querySelectorAll('#posItemsBody tr').forEach(row => {
        const name = row.querySelector('.pos-name').value.trim();
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = isInv ? 0 : (parseFloat(row.querySelector('.pos-cost').value) || 0);
        const price = isInv ? 0 : (parseFloat(row.querySelector('.pos-price').value) || 0);
        totalCost += (qty * cost);
        if (name) {
          productSummary.push(`${name} (x${qty})`);
          itemsList.push({ name, qty, cost, price, subtotal: qty * price });
        }
      });

      transactions.push({ id: Date.now(), date, customer: cust, location: loc, product: productSummary.join(', '), itemsList, total, totalCost, paid, balance, status, payments });
      saveData();
      alert('Transaction saved successfully!');
      this.reset();
      document.getElementById('posItemsBody').innerHTML = '';
      document.getElementById('inventoryOnlyMode').checked = false;
      toggleInventoryOnlyMode();
      addPosRow();
      document.getElementById('saleDate').value = getTodayDateString();
      generateDailyReport();
    });

    function generateDailyReport() {
      const date = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';
      let totalSales = 0, totalCollected = 0, totalNet = 0, count = 0;

      transactions.forEach((t, index) => {
        if (t.date === date) {
          totalSales += t.total;
          let netProf = t.total - (t.totalCost || 0);
          totalNet += netProf;
          count++;

          let methodStr = t.payments ? t.payments.map(p => `${p.method}: ₱${p.amount.toFixed(2)}`).join(', ') : 'Cash';

          tbody.innerHTML += `
            <tr>
              <td>${index+1}</td>
              <td class="fw-bold">${t.customer}</td>
              <td><span class="badge bg-secondary">${t.location}</span></td>
              <td>${t.product}</td>
              <td>${t.containerInfo || 'Wala'}</td>
              <td>₱${(t.totalCost||0).toFixed(2)}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger">₱${t.balance.toFixed(2)}</td>
              <td class="text-success fw-bold">₱${netProf.toFixed(2)}</td>
              <td><small>${methodStr}</small></td>
              <td class="no-print text-center">
                <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteTransaction(${t.id})" title="Delete"><i class="fa-solid fa-trash-can"></i></button>
              </td>
            </tr>
          `;
        }
        if (t.payments) {
          t.payments.forEach(p => { if (p.date === date) totalCollected += p.amount; });
        }
      });

      if(count === 0) {
        tbody.innerHTML = `<tr><td colspan="12" class="text-center text-muted py-3">Wala pang transaksyon sa araw na ito.</td></tr>`;
      }

      document.getElementById('dailyTotalSales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${totalCollected.toFixed(2)}`;
      document.getElementById('dailyTotalNetProfit').innerText = `₱${totalNet.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;
    }

    function deleteTransaction(id) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang transaksyong ito?')) {
        transactions = transactions.filter(t => t.id !== id);
        saveData();
        generateDailyReport();
        renderCreditTable();
      }
    }

    function renderCreditTable() {
      const tbody = document.getElementById('creditTableBody');
      tbody.innerHTML = '';
      transactions.forEach(t => {
        if (t.balance > 0.01) {
          tbody.innerHTML += `<tr><td>${t.customer}</td><td>${t.location}</td><td>${t.product}</td><td>₱${t.totalCost.toFixed(2)}</td><td>₱${t.paid.toFixed(2)}</td><td>₱${t.balance.toFixed(2)}</td><td>${t.dueDate||'N/A'}</td><td>${t.status}</td><td></td></tr>`;
        }
      });
    }

    function searchCustomerOrder() {
      const query = document.getElementById('searchCustomerInput').value.trim().toLowerCase();
      const container = document.getElementById('searchResultContainer');
      const notFound = document.getElementById('noCustomerFound');
      if (!query) { container.style.display = 'none'; notFound.classList.add('d-none'); return; }
      const matched = transactions.filter(t => t.customer.toLowerCase().includes(query));
      if (matched.length > 0) {
        notFound.classList.add('d-none');
        container.style.display = 'block';
        document.getElementById('lastOrderCustomer').innerText = matched[matched.length - 1].customer;
      } else {
        container.style.display = 'none';
        notFound.classList.remove('d-none');
      }
    }
  </script>
</body>
</html>
