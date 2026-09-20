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
    .nav-pills .nav-link:hover { background-color: rgba(255,255,255,0.2); }
    .credit-fields { display: none; background-color: #f8f9fa; border-radius: 8px; padding: 15px; margin-top: 15px; border: 1px dashed #cbd5e1; }
    
    .col-action { width: 45px; text-align: center; vertical-align: middle; }
    .inventory-input { width: 75px; text-align: center; }
    .stat-card { border-left: 4px solid #1976d2; }
  </style>
</head>
<body>

  <!-- Navbar -->
  <nav class="navbar navbar-dark expand-lg mb-4">
    <div class="container-fluid">
      <a class="navbar-brand fw-bold fs-4" href="#">
        <i class="fa-solid fa-store me-2"></i>RMVillasis Enterprises POS
      </a>
      <ul class="nav nav-pills me-auto" id="mainTabs" role="tablist">
        <li class="nav-item">
          <button class="nav-link active" id="pos-tab" data-bs-toggle="pill" data-bs-target="#pos-content" type="button">
            <i class="fa-solid fa-cart-shopping me-1"></i> POS Entry
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="daily-tab" data-bs-toggle="pill" data-bs-target="#daily-content" type="button" onclick="generateDailyReport()">
            <i class="fa-solid fa-calendar-day me-1"></i> Daily Report
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button" onclick="renderCreditTable()">
            <i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTable()">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()">
            <i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit & Net Profit
          </button>
        </li>
      </ul>
    </div>
  </nav>

  <div class="container pb-5">
    <div class="tab-content" id="mainTabsContent">

      <!-- ================= 1. POS ENTRY TAB ================= -->
      <div class="tab-pane fade show active" id="pos-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-cash-register me-2"></i>Record New Transaction</h4>
          <form id="posForm">
            <div class="row g-3 mb-4">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Date of Sale:</label>
                <input type="date" id="saleDate" class="form-control" required>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Customer Name:</label>
                <input type="text" id="customerName" class="form-control" placeholder="e.g., Juan Dela Cruz" required>
              </div>
            </div>

            <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-cart-plus me-2"></i>Items / Products</h6>
            <div class="table-responsive mb-3">
              <table class="table table-bordered align-middle" id="posItemsTable">
                <thead class="table-light">
                  <tr>
                    <th>Product Name</th>
                    <th style="width: 120px;">Qty</th>
                    <th style="width: 150px;">Cost / Unit (₱)</th>
                    <th style="width: 150px;">Price / Unit (₱)</th>
                    <th style="width: 150px;">Subtotal (₱)</th>
                    <th class="col-action"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="posItemsBody">
                  <!-- Dynamic Rows -->
                </tbody>
              </table>
              <button type="button" class="btn btn-sm btn-outline-primary" onclick="addPosRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Another Item
              </button>
            </div>

            <div class="row g-3 mt-2">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="number" step="0.01" id="totalAmount" class="form-control bg-light fs-5 fw-bold text-primary" readonly placeholder="0.00">
              </div>

              <div class="col-md-4">
                <label class="form-label fw-semibold">Transaction Type:</label>
                <select id="paymentType" class="form-select" onchange="toggleCreditFields()">
                  <option value="FULL">Paid in Full (Buong Bayad)</option>
                  <option value="PARTIAL">Partial Payment (May Natirang Utang)</option>
                  <option value="CREDIT">Full Credit / Purong Utang</option>
                </select>
              </div>

              <div class="col-md-4">
                <label class="form-label fw-semibold">Payment Method:</label>
                <select id="paymentMethod" class="form-select">
                  <option value="Cash">Cash</option>
                  <option value="GCash">GCash</option>
                  <option value="Bank Transfer">Bank Transfer (BT)</option>
                  <option value="Cheque">Cheque</option>
                </select>
              </div>
            </div>

            <div id="creditFieldsSection" class="credit-fields">
              <h6 class="text-secondary fw-bold mb-3"><i class="fa-solid fa-file-invoice-dollar me-2"></i>Utang / Credit Details</h6>
              <div class="row g-3">
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Amount Paid Now (₱):</label>
                  <input type="number" step="0.01" id="amountPaidNow" class="form-control" value="0.00" oninput="calculateBalance()">
                </div>
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Remaining Balance (₱):</label>
                  <input type="number" step="0.01" id="remainingBalance" class="form-control bg-light" readonly value="0.00">
                </div>
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Due Date:</label>
                  <input type="date" id="dueDate" class="form-control">
                </div>
              </div>
            </div>

            <div class="mt-4 text-end">
              <button type="submit" class="btn btn-success btn-lg px-4"><i class="fa-solid fa-check me-2"></i>Save Transaction</button>
            </div>
          </form>
        </div>
      </div>

      <!-- ================= 2. DAILY REPORT TAB ================= -->
      <div class="tab-pane fade" id="daily-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-calendar-day me-2"></i>Daily Sales & Encoded Logs</h4>
            <div class="d-flex gap-2 align-items-center">
              <label class="fw-bold me-1">Select Date:</label>
              <input type="date" id="dailyReportDate" class="form-control" onchange="generateDailyReport()">
            </div>
          </div>

          <!-- Daily Overview Cards -->
          <div class="row g-3 mb-4">
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">DAILY TOTAL SALES</span>
                <h4 class="text-primary mt-1 mb-0" id="dailyTotalSales">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">DAILY CASH / COLLECTION</span>
                <h4 class="text-success mt-1 mb-0" id="dailyTotalCollected">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;">
                <span class="text-muted small fw-bold">TRANSACTIONS ENCODED</span>
                <h4 class="text-warning mt-1 mb-0" id="dailyTxCount">0</h4>
              </div>
            </div>
          </div>

          <!-- Daily Encoded Table -->
          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>List of Encoded Transactions Today</h6>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>#</th>
                  <th>Customer Name</th>
                  <th>Products Bought</th>
                  <th>Total Amount</th>
                  <th>Paid Amount</th>
                  <th>Balance</th>
                  <th>Payment Method</th>
                </tr>
              </thead>
              <tbody id="dailyTableBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 3. UTANG & PAYMENTS TAB ================= -->
      <div class="tab-pane fade" id="credit-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-users-viewfinder me-2"></i>Customer Credit & Utang Ledger</h4>
          <div class="table-responsive">
            <table class="table table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>Customer Name</th>
                  <th>Product(s)</th>
                  <th>Total Cost (₱)</th>
                  <th>Amount Paid (₱)</th>
                  <th>Balance (₱)</th>
                  <th>Due Date</th>
                  <th>Status</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody id="creditTableBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 4. INVENTORY TAB ================= -->
      <div class="tab-pane fade" id="inventory-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory Management</h4>
            <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#addProductModal">
              <i class="fa-solid fa-plus me-1"></i> Add Product
            </button>
          </div>

          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark text-center">
                <tr>
                  <th class="text-start">Product Name</th>
                  <th>Qty / Unit Stock</th>
                  <th>Beginning Stock</th>
                  <th>Stock In (+Add)</th>
                  <th>Sold</th>
                  <th>Ending Stock</th>
                  <th class="col-action"><i class="fa-solid fa-trash"></i></th>
                </tr>
              </thead>
              <tbody id="inventoryTableBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 5. MONTHLY AUDIT & NET PROFIT TAB ================= -->
      <div class="tab-pane fade" id="audit-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-chart-pie me-2"></i>Monthly Audit & Net Profit</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-warning fw-bold text-dark me-2" data-bs-toggle="modal" data-bs-target="#bossModal">
                <i class="fa-solid fa-user-tie me-1"></i> Add D/Eco Boss Adjustment
              </button>
              <label class="fw-bold me-1">Filter Month:</label>
              <input type="month" id="auditMonth" class="form-control" onchange="generateMonthlyAudit()">
            </div>
          </div>

          <!-- Audit Overview Cards -->
          <div class="row g-3 mb-4">
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">TOTAL GROSS SALES</span>
                <h4 class="text-primary mt-1 mb-0" id="auditTotalSales">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">TOTAL CASH COLLECTED</span>
                <h4 class="text-success mt-1 mb-0" id="auditTotalCollected">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #c62828;">
                <span class="text-muted small fw-bold">TOTAL SALARIES / EXPENSES</span>
                <h4 class="text-danger mt-1 mb-0" id="auditExpenses">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #0d47a1;">
                <span class="text-muted small fw-bold">NET PROFIT (Incl. Boss)</span>
                <h4 class="text-primary fw-bold mt-1 mb-0" id="auditNetProfit">₱0.00</h4>
              </div>
            </div>
          </div>

          <!-- DETAILED SALARY & EXPENSES SECTION -->
          <div class="card p-3 bg-light mb-4 border">
            <div class="d-flex justify-content-between align-items-center mb-2">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-receipt me-2"></i>Itemized Salary & Expenses Breakdown</h6>
              <button class="btn btn-sm btn-outline-danger" onclick="addExpenseRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Expense Line
              </button>
            </div>
            <div class="table-responsive">
              <table class="table table-bordered align-middle bg-white" id="expenseTable">
                <thead class="table-light">
                  <tr>
                    <th>Expense Name / Description</th>
                    <th style="width: 200px;">Amount (₱)</th>
                    <th class="col-action"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="expenseTableBody">
                  <!-- Dynamic Expense Rows -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- D/Eco Boss Table Log -->
          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Transactions Log</h6>
          <div class="table-responsive mb-4">
            <table class="table table-sm table-bordered bg-white">
              <thead class="table-light">
                <tr>
                  <th>Date</th>
                  <th>Description / Type</th>
                  <th>Amount (₱)</th>
                </tr>
              </thead>
              <tbody id="bossLogsBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>

        </div>
      </div>

    </div>
  </div>

  <!-- Modal para sa D/Eco Boss Adjustment -->
  <div class="modal fade" id="bossModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-warning">
          <h5 class="modal-title fw-bold text-dark"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Adjustment</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <form id="bossForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date:</label>
              <input type="date" id="bossDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Adjustment Type:</label>
              <select id="bossType" class="form-select">
                <option value="ADD"> Boss Addition / Capital Cash In (+ Net Profit)</option>
                <option value="SUB"> Boss Withdrawal / Cash Out (- Net Profit)</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Amount (₱):</label>
              <input type="number" step="0.01" id="bossAmount" class="form-control" placeholder="0.00" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Notes / Description:</label>
              <input type="text" id="bossNotes" class="form-control" placeholder="e.g., Personal Withdrawal, Additional Capital">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-dark"><i class="fa-solid fa-floppy-disk me-1"></i>Save Adjustment</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- Modal para sa Pagbabayad ng Utang -->
  <div class="modal fade" id="paymentModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-hand-holding-dollar me-2"></i>Record Utang Payment</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <input type="hidden" id="payTransactionId">
          <div class="mb-3">
            <label class="form-label fw-semibold">Customer Name:</label>
            <input type="text" id="payCustomerName" class="form-control bg-light" readonly>
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Current Balance (₱):</label>
            <input type="number" id="payCurrentBalance" class="form-control bg-light" readonly>
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold text-success">Amount Paying Now (₱):</label>
            <input type="number" step="0.01" id="payAmountInput" class="form-control" placeholder="0.00" required>
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Payment Method:</label>
            <select id="payMethod" class="form-select">
              <option value="Cash">Cash</option>
              <option value="GCash">GCash</option>
              <option value="Bank Transfer">Bank Transfer (BT)</option>
              <option value="Cheque">Cheque</option>
            </select>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
          <button type="button" class="btn btn-success" onclick="submitUtangPayment()"><i class="fa-solid fa-floppy-disk me-1"></i>Save Payment</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal para sa Pagdaragdag ng Bagong Produkto -->
  <div class="modal fade" id="addProductModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-box-open me-2"></i>Add New Product</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="addProductForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name:</label>
              <input type="text" id="newProdName" class="form-control" placeholder="e.g., Semento" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Qty / Unit Stock:</label>
              <input type="number" id="newProdQty" class="form-control" value="1" min="1" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Beginning Stock:</label>
              <input type="number" id="newProdStock" class="form-control" value="0" min="0" required>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk me-1"></i>Save Product</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    // Database Objects
    let transactions = [
      { id: 1, date: "2026-09-20", customer: "Juan Dela Cruz", product: "Semento (x1)", total: 250.00, paid: 100.00, balance: 150.00, dueDate: "2026-09-30", status: "PARTIAL", payments: [{ amount: 100.00, method: "Cash", date: "2026-09-20" }] },
      { id: 2, date: "2026-09-20", customer: "Maria Clara", product: "Pintura Red (x1)", total: 450.00, paid: 450.00, balance: 0.00, dueDate: "N/A", status: "PAID", payments: [{ amount: 450.00, method: "GCash", date: "2026-09-20" }] }
    ];

    let inventory = [
      { id: 1, name: "Semento (1 Bag)", qty: 100, beginning: 100, stockIn: 20, ending: 80 },
      { id: 2, name: "Pintura Red", qty: 50, beginning: 50, stockIn: 0, ending: 45 }
    ];

    let bossAdjustments = [];
    
    // Structure for detailed expenses per month: { "YYYY-MM": [ { name: "", amount: 0 } ] }
    let monthlyExpensesData = {
      "2026-09": [
        { name: "Sahod ni Juan", amount: 5000.00 },
        { name: "Kuryente at Tubig", amount: 1500.00 }
      ]
    };

    // Initial Date Configurations
    const today = new Date();
    const todayFormatted = today.toISOString().split('T')[0];
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;
    document.getElementById('bossDate').value = todayFormatted;
    document.getElementById('auditMonth').value = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}`;

    // ================= POS LOGIC =================
    function addPosRow() {
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now();
      const rowHTML = `
        <tr id="row-${rowId}">
          <td><input type="text" class="form-control form-control-sm pos-name" placeholder="Product Name" required></td>
          <td><input type="number" class="form-control form-control-sm pos-qty" value="1" min="1" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-price" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm bg-light pos-subtotal" placeholder="0.00" readonly></td>
          <td class="col-action">
            <button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="removePosRow('row-${rowId}')">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </td>
        </tr>
      `;
      tbody.insertAdjacentHTML('beforeend', rowHTML);
      calculateTotal();
    }

    function removePosRow(rowId) {
      const rows = document.querySelectorAll('#posItemsBody tr');
      if (rows.length > 1) {
        document.getElementById(rowId).remove();
        calculateTotal();
      } else {
        alert('Kailangang mayroong kahit isang produkto sa resibo.');
      }
    }

    function calculateTotal() {
      const rows = document.querySelectorAll('#posItemsBody tr');
      let grandTotal = 0;
      rows.forEach(row => {
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const price = parseFloat(row.querySelector('.pos-price').value) || 0;
        const subtotal = qty * price;
        row.querySelector('.pos-subtotal').value = subtotal.toFixed(2);
        grandTotal += subtotal;
      });
      document.getElementById('totalAmount').value = grandTotal.toFixed(2);
      calculateBalance();
    }

    function toggleCreditFields() {
      const paymentType = document.getElementById('paymentType').value;
      const creditSection = document.getElementById('creditFieldsSection');
      if (paymentType === 'FULL') {
        creditSection.style.display = 'none';
        document.getElementById('amountPaidNow').value = document.getElementById('totalAmount').value;
      } else {
        creditSection.style.display = 'block';
        if (paymentType === 'CREDIT') {
          document.getElementById('amountPaidNow').value = '0.00';
        }
      }
      calculateBalance();
    }

    function calculateBalance() {
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paymentType = document.getElementById('paymentType').value;
      let paidNow = parseFloat(document.getElementById('amountPaidNow').value) || 0;

      if (paymentType === 'FULL') paidNow = total;
      if (paymentType === 'CREDIT') paidNow = 0;

      const balance = Math.max(0, total - paidNow);
      document.getElementById('remainingBalance').value = balance.toFixed(2);
    }

    document.getElementById('posForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paymentType = document.getElementById('paymentType').value;
      const method = document.getElementById('paymentMethod').value;
      const saleDate = document.getElementById('saleDate').value;
      let paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
      
      if (paymentType === 'FULL') paid = total;
      if (paymentType === 'CREDIT') paid = 0;

      const balance = total - paid;
      let status = "PAID";
      if (balance > 0 && paid > 0) status = "PARTIAL";
      if (balance > 0 && paid === 0) status = "UNPAID";

      const itemRows = document.querySelectorAll('#posItemsBody tr');
      let productSummary = [];
      itemRows.forEach(row => {
        const name = row.querySelector('.pos-name').value;
        const qty = row.querySelector('.pos-qty').value;
        if(name) productSummary.push(`${name} (x${qty})`);
      });

      const paymentHistory = [];
      if (paid > 0) {
        paymentHistory.push({ amount: paid, method: method, date: saleDate });
      }

      transactions.push({
        id: transactions.length + 1,
        date: saleDate,
        customer: document.getElementById('customerName').value,
        product: productSummary.join(', '),
        total: total,
        paid: paid,
        balance: balance,
        dueDate: document.getElementById('dueDate').value || 'N/A',
        status: status,
        payments: paymentHistory
      });

      alert('Transaction saved successfully!');
      this.reset();
      document.getElementById('posItemsBody').innerHTML = '';
      addPosRow();
      document.getElementById('saleDate').value = todayFormatted;
      toggleCreditFields();
    });

    // ================= DAILY REPORT LOGIC =================
    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';

      let daySales = 0;
      let dayCollected = 0;
      let count = 0;

      const filtered = transactions.filter(t => t.date === selectedDate);

      filtered.forEach((t, index) => {
        daySales += t.total;
        dayCollected += t.paid;
        count++;

        tbody.innerHTML += `
          <tr>
            <td>${index + 1}</td>
            <td class="fw-bold">${t.customer}</td>
            <td>${t.product}</td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger">₱${t.balance.toFixed(2)}</td>
            <td>${t.payments.length > 0 ? t.payments[0].method : 'N/A'}</td>
          </tr>
        `;
      });

      if (filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="7" class="text-center text-muted py-3">Walang na-encode na transaksyon sa petsang ito.</td></tr>`;
      }

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;
    }

    // ================= D/ECO BOSS LOGIC =================
    document.getElementById('bossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('bossDate').value;
      const type = document.getElementById('bossType').value;
      const amount = parseFloat(document.getElementById('bossAmount').value) || 0;
      const notes = document.getElementById('bossNotes').value || 'D/Eco Boss Adjustment';

      bossAdjustments.push({ date, type, amount, notes });

      alert('D/Eco Boss Adjustment saved!');
      bootstrap.Modal.getInstance(document.getElementById('bossModal')).hide();
      this.reset();
      generateMonthlyAudit();
    });

    // ================= DETAILED EXPENSES LOGIC =================
    function addExpenseRow(name = '', amount = '') {
      const tbody = document.getElementById('expenseTableBody');
      const rowId = Date.now();
      const rowHTML = `
        <tr id="exp-${rowId}">
          <td><input type="text" class="form-control form-control-sm exp-name" placeholder="e.g., Sahod ni Juan, Kuryente, etc." value="${name}" oninput="saveCurrentMonthExpenses()"></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm exp-amount" placeholder="0.00" value="${amount}" oninput="saveCurrentMonthExpenses()"></td>
          <td class="col-action">
            <button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="removeExpenseRow('exp-${rowId}')">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </td>
        </tr>
      `;
      tbody.insertAdjacentHTML('beforeend', rowHTML);
      saveCurrentMonthExpenses();
    }

    function removeExpenseRow(rowId) {
      document.getElementById(rowId).remove();
      saveCurrentMonthExpenses();
    }

    function saveCurrentMonthExpenses() {
      const selectedMonth = document.getElementById('auditMonth').value;
      const rows = document.querySelectorAll('#expenseTableBody tr');
      let expList = [];

      rows.forEach(row => {
        const name = row.querySelector('.exp-name').value;
        const amount = parseFloat(row.querySelector('.exp-amount').value) || 0;
        expList.push({ name, amount });
      });

      monthlyExpensesData[selectedMonth] = expList;
      generateMonthlyAudit();
    }

    // ================= MONTHLY AUDIT & NET PROFIT =================
    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('auditMonth').value; // YYYY-MM
      if (!selectedMonth) return;

      let grossSales = 0;
      let totalCollected = 0;

      transactions.forEach(tx => {
        if (tx.date.startsWith(selectedMonth)) {
          grossSales += tx.total;
        }
        tx.payments.forEach(p => {
          if (p.date.startsWith(selectedMonth)) {
            totalCollected += p.amount;
          }
        });
      });

      // Render Expense Table Rows for Selected Month
      const expBody = document.getElementById('expenseTableBody');
      expBody.innerHTML = '';
      const monthExp = monthlyExpensesData[selectedMonth] || [];

      if (monthExp.length === 0) {
        // If empty, add default single row
        addExpenseRow();
      } else {
        monthExp.forEach(exp => {
          const rowId = Date.now() + Math.random();
          expBody.innerHTML += `
            <tr id="exp-${rowId}">
              <td><input type="text" class="form-control form-control-sm exp-name" placeholder="Expense Name" value="${exp.name}" oninput="saveCurrentMonthExpenses()"></td>
              <td><input type="number" step="0.01" class="form-control form-control-sm exp-amount" placeholder="0.00" value="${exp.amount || ''}" oninput="saveCurrentMonthExpenses()"></td>
              <td class="col-action">
                <button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="removeExpenseRow('exp-${rowId}')">
                  <i class="fa-solid fa-trash-can"></i>
                </button>
              </td>
            </tr>
          `;
        });
      }

      // Calculate Total Expenses
      let totalExpenses = 0;
      (monthlyExpensesData[selectedMonth] || []).forEach(e => {
        totalExpenses += e.amount;
      });

      // Boss Adjustments for selected month
      let bossTotal = 0;
      const bossBody = document.getElementById('bossLogsBody');
      bossBody.innerHTML = '';

      bossAdjustments.filter(b => b.date.startsWith(selectedMonth)).forEach(b => {
        const val = b.type === 'ADD' ? b.amount : -b.amount;
        bossTotal += val;

        bossBody.innerHTML += `
          <tr>
            <td>${b.date}</td>
            <td>${b.notes} (${b.type === 'ADD' ? 'Capital In' : 'Withdrawal'})</td>
            <td class="${b.type === 'ADD' ? 'text-success' : 'text-danger'} fw-bold">
              ${b.type === 'ADD' ? '+' : '-'}₱${b.amount.toFixed(2)}
            </td>
          </tr>
        `;
      });

      if (bossBody.innerHTML === '') {
        bossBody.innerHTML = `<tr><td colspan="3" class="text-center text-muted">Walang na-record na D/Eco Boss adjustment sa buwang ito.</td></tr>`;
      }

      // Calculation: Net Profit = Total Collection - Expenses + Boss Adjustments
      const netProfit = totalCollected - totalExpenses + bossTotal;

      document.getElementById('auditTotalSales').innerText = `₱${grossSales.toFixed(2)}`;
      document.getElementById('auditTotalCollected').innerText = `₱${totalCollected.toFixed(2)}`;
      document.getElementById('auditExpenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${netProfit.toFixed(2)}`;
    }

    // ================= UTANG & PAYMENTS =================
    function renderCreditTable() {
      const tbody = document.getElementById('creditTableBody');
      tbody.innerHTML = '';
      const utangList = transactions.filter(t => t.balance > 0);

      if (utangList.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Walang nakatalang may utang sa kasalukuyan.</td></tr>`;
        return;
      }

      utangList.forEach(t => {
        const statusBadge = t.status === 'UNPAID' 
          ? `<span class="badge bg-danger">UNPAID</span>` 
          : `<span class="badge bg-warning text-dark">PARTIAL</span>`;

        tbody.innerHTML += `
          <tr>
            <td class="fw-bold">${t.customer}</td>
            <td>${t.product}</td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
            <td>${t.dueDate}</td>
            <td>${statusBadge}</td>
            <td>
              <button class="btn btn-sm btn-primary" onclick="openPaymentModal(${t.id})">
                <i class="fa-solid fa-receipt me-1"></i> Pay
              </button>
            </td>
          </tr>
        `;
      });
    }

    let selectedModal;
    function openPaymentModal(txId) {
      const tx = transactions.find(t => t.id === txId);
      if (!tx) return;
      document.getElementById('payTransactionId').value = tx.id;
      document.getElementById('payCustomerName').value = tx.customer;
      document.getElementById('payCurrentBalance').value = tx.balance.toFixed(2);
      document.getElementById('payAmountInput').value = '';
      selectedModal = new bootstrap.Modal(document.getElementById('paymentModal'));
      selectedModal.show();
    }

    function submitUtangPayment() {
      const txId = parseInt(document.getElementById('payTransactionId').value);
      const payAmount = parseFloat(document.getElementById('payAmountInput').value) || 0;
      const method = document.getElementById('payMethod').value;
      const tx = transactions.find(t => t.id === txId);

      if (payAmount <= 0 || payAmount > tx.balance) {
        alert('Mangyaring maglagay ng tamang halaga ng ibabayad.');
        return;
      }

      tx.paid += payAmount;
      tx.balance -= payAmount;
      tx.status = tx.balance <= 0 ? 'PAID' : 'PARTIAL';
      
      tx.payments.push({
        amount: payAmount,
        method: method,
        date: new Date().toISOString().split('T')[0]
      });

      alert('Na-record nang matagumpay ang bayad!');
      selectedModal.hide();
      renderCreditTable();
    }

    // ================= INVENTORY =================
    function renderInventoryTable() {
      const tbody = document.getElementById('inventoryTableBody');
      tbody.innerHTML = '';
      if (inventory.length === 0) {
        tbody.innerHTML = `<tr><td colspan="7" class="text-center text-muted py-3">Walang laman ang inventory.</td></tr>`;
        return;
      }

      inventory.forEach((item, index) => {
        const sold = Math.max(0, (item.beginning + item.stockIn) - item.ending);
        tbody.innerHTML += `
          <tr>
            <td><input type="text" class="form-control form-control-sm fw-bold" value="${item.name}" onchange="updateInventory(${index}, 'name', this.value)"></td>
            <td><input type="number" class="form-control form-control-sm text-center mx-auto inventory-input" value="${item.qty}" min="0" onchange="updateInventory(${index}, 'qty', this.value)"></td>
            <td><input type="number" class="form-control form-control-sm text-center mx-auto inventory-input" value="${item.beginning}" min="0" onchange="updateInventory(${index}, 'beginning', this.value)"></td>
            <td><input type="number" class="form-control form-control-sm text-center mx-auto inventory-input" value="${item.stockIn}" min="0" onchange="updateInventory(${index}, 'stockIn', this.value)"></td>
            <td class="text-center align-middle fw-semibold text-primary">${sold}</td>
            <td><input type="number" class="form-control form-control-sm text-center mx-auto inventory-input" value="${item.ending}" min="0" onchange="updateInventory(${index}, 'ending', this.value)"></td>
            <td class="col-action align-middle">
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteInventoryItem(${index})">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;
      });
    }

    function updateInventory(index, key, value) {
      inventory[index][key] = key === 'name' ? value : (parseInt(value) || 0);
      renderInventoryTable();
    }

    function deleteInventoryItem(index) {
      if (confirm('Sigurado ka bang gusto mong burahin ang item na ito?')) {
        inventory.splice(index, 1);
        renderInventoryTable();
      }
    }

    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      inventory.push({
        id: inventory.length + 1,
        name: document.getElementById('newProdName').value,
        qty: parseInt(document.getElementById('newProdQty').value) || 1,
        beginning: parseInt(document.getElementById('newProdStock').value) || 0,
        stockIn: 0,
        ending: parseInt(document.getElementById('newProdStock').value) || 0
      });
      this.reset();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      renderInventoryTable();
    });

    // Initializations
    addPosRow();
    generateDailyReport();
  </script>
</body>
</html>
