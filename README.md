<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>RMVillasis Enterprises POS</title>
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
          <button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button">
            <i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="sales-tab" data-bs-toggle="pill" data-bs-target="#sales-content" type="button">
            <i class="fa-solid fa-chart-line me-1"></i> Sales Management
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
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-cash-register me-2"></i>Record New Sale</h4>
          <form id="posForm">
            <div class="row g-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Date of Sale:</label>
                <input type="date" id="saleDate" class="form-control" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer Name:</label>
                <input type="text" id="customerName" class="form-control" placeholder="e.g., Juan Dela Cruz" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Product Name:</label>
                <input type="text" id="productName" class="form-control" placeholder="Item description" required>
              </div>

              <div class="col-md-4">
                <label class="form-label fw-semibold">Quantity:</label>
                <input type="number" id="quantity" class="form-control" value="1" min="1" oninput="calculateTotal()" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Buying Price / Cost (₱):</label>
                <input type="number" step="0.01" id="buyingPrice" class="form-control" placeholder="0.00" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Selling Price (₱):</label>
                <input type="number" step="0.01" id="sellingPrice" class="form-control" placeholder="0.00" oninput="calculateTotal()" required>
              </div>

              <div class="col-md-6">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="number" step="0.01" id="totalAmount" class="form-control bg-light" readonly placeholder="0.00">
              </div>

              <div class="col-md-6">
                <label class="form-label fw-semibold">Payment Type:</label>
                <select id="paymentType" class="form-select" onchange="toggleCreditFields()">
                  <option value="FULL">Cash / Paid in Full</option>
                  <option value="PARTIAL">Partial Payment (May Natirang Utang)</option>
                  <option value="CREDIT">Full Credit / Purong Utang</option>
                </select>
              </div>
            </div>

            <!-- Conditional Credit Fields -->
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

      <!-- ================= 2. UTANG & PAYMENTS TAB ================= -->
      <div class="tab-pane fade" id="credit-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-users-viewfinder me-2"></i>Customer Credit & Utang Ledger</h4>
          </div>
          
          <div class="table-responsive">
            <table class="table table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>Customer Name</th>
                  <th>Product / Item</th>
                  <th>Total Cost (₱)</th>
                  <th>Amount Paid (₱)</th>
                  <th>Balance (₱)</th>
                  <th>Due Date</th>
                  <th>Status</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody id="creditTableBody">
                <!-- Dynamic Content via JS -->
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 3. INVENTORY TAB ================= -->
      <div class="tab-pane fade" id="inventory-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory Management</h4>
          <p class="text-muted">Talahanayan ng stocks at mga paninda.</p>
        </div>
      </div>

      <!-- ================= 4. SALES MANAGEMENT TAB ================= -->
      <div class="tab-pane fade" id="sales-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-receipt me-2"></i>Sales History</h4>
          <p class="text-muted">Talaan ng lahat ng nakaraang bentahan.</p>
        </div>
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
              <option value="Bank Transfer">Bank Transfer</option>
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

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    // Sample In-Memory Database
    let transactions = [
      { id: 1, date: "2026-09-20", customer: "Juan Dela Cruz", product: "Semento (1 Bag)", total: 250.00, paid: 100.00, balance: 150.00, dueDate: "2026-09-30", status: "PARTIAL" },
      { id: 2, date: "2026-09-19", customer: "Maria Clara", product: "Pintura Red", total: 450.00, paid: 0.00, balance: 450.00, dueDate: "2026-09-28", status: "UNPAID" }
    ];

    // Set Default Date to Today
    document.getElementById('saleDate').valueAsDate = new Date();

    function calculateTotal() {
      const qty = parseFloat(document.getElementById('quantity').value) || 0;
      const price = parseFloat(document.getElementById('sellingPrice').value) || 0;
      const total = qty * price;
      document.getElementById('totalAmount').value = total.toFixed(2);
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

      if (paymentType === 'FULL') {
        paidNow = total;
      } else if (paymentType === 'CREDIT') {
        paidNow = 0;
      }

      const balance = Math.max(0, total - paidNow);
      document.getElementById('remainingBalance').value = balance.toFixed(2);
    }

    // Save New Transaction
    document.getElementById('posForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paymentType = document.getElementById('paymentType').value;
      let paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
      
      if (paymentType === 'FULL') paid = total;
      if (paymentType === 'CREDIT') paid = 0;

      const balance = total - paid;
      let status = "PAID";
      if (balance > 0 && paid > 0) status = "PARTIAL";
      if (balance > 0 && paid === 0) status = "UNPAID";

      const newTx = {
        id: transactions.length + 1,
        date: document.getElementById('saleDate').value,
        customer: document.getElementById('customerName').value,
        product: document.getElementById('productName').value,
        total: total,
        paid: paid,
        balance: balance,
        dueDate: document.getElementById('dueDate').value || 'N/A',
        status: status
      };

      transactions.push(newTx);
      alert('Transaction saved successfully!');
      this.reset();
      document.getElementById('saleDate').valueAsDate = new Date();
      toggleCreditFields();
      renderCreditTable();
    });

    // Render Utang Table
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

    // Modal Control for Utang Payment
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
      const tx = transactions.find(t => t.id === txId);

      if (payAmount <= 0) {
        alert('Mangyaring maglagay ng tamang halaga ng ibabayad.');
        return;
      }

      if (payAmount > tx.balance) {
        alert('Ang ibinabayad ay higit sa natitirang utang!');
        return;
      }

      tx.paid += payAmount;
      tx.balance -= payAmount;

      if (tx.balance <= 0) {
        tx.balance = 0;
        tx.status = 'PAID';
      } else {
        tx.status = 'PARTIAL';
      }

      alert('Na-record nang matagumpay ang bayad!');
      selectedModal.hide();
      renderCreditTable();
    }

    // Initial Render
    renderCreditTable();
  </script>
</body>
</html>
