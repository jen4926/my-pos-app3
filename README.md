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
    .credit-fields, .container-fields, .multi-payment-fields { display: none; background-color: #f8f9fa; border-radius: 8px; padding: 15px; margin-top: 15px; border: 1px dashed #cbd5e1; }
    .col-action { width: 45px; text-align: center; vertical-align: middle; }
    .stat-card { border-left: 4px solid #1976d2; }
    @media print {
      body { background-color: #fff !important; color: #000 !important; font-size: 12pt; }
      .navbar, .btn, .nav, .modal, .no-print { display: none !important; }
      .card { border: none !important; box-shadow: none !important; padding: 0 !important; }
      .container { max-width: 100% !important; padding: 0 !important; margin: 0 !important; }
      .tab-pane { display: block !important; opacity: 1 !important; }
    }
  </style>
</head>
<body>

  <!-- Navbar -->
  <nav class="navbar navbar-dark expand-lg mb-4">
    <div class="container-fluid">
      <a class="navbar-brand fw-bold fs-4" href="#"><i class="fa-solid fa-store me-2"></i>RMVillasis Enterprises</a>
      <ul class="nav nav-pills me-auto" id="mainTabs" role="tablist">
        <li class="nav-item"><button class="nav-link active" id="pos-tab" data-bs-toggle="pill" data-bs-target="#pos-content" type="button"><i class="fa-solid fa-cart-shopping me-1"></i> POS Entry</button></li>
        <li class="nav-item"><button class="nav-link" id="manual-payment-tab" data-bs-toggle="pill" data-bs-target="#manual-payment-content" type="button"><i class="fa-solid fa-pen-to-square me-1"></i> Manual Payment</button></li>
        <li class="nav-item"><button class="nav-link" id="backup-tab" data-bs-toggle="pill" data-bs-target="#backup-content" type="button"><i class="fa-solid fa-shield-halved me-1"></i> Data Backup & Safety</button></li>
        <li class="nav-item"><button class="nav-link" id="daily-tab" data-bs-toggle="pill" data-bs-target="#daily-content" type="button" onclick="generateDailyReport()"><i class="fa-solid fa-calendar-day me-1"></i> Daily Report</button></li>
        <li class="nav-item"><button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button" onclick="renderCreditTable();"><i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments</button></li>
        <li class="nav-item"><button class="nav-link" id="search-tab" data-bs-toggle="pill" data-bs-target="#search-content" type="button"><i class="fa-solid fa-magnifying-glass me-1"></i> Order Lookup & Last Price</button></li>
        <li class="nav-item"><button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTables()"><i class="fa-solid fa-boxes-stacked me-1"></i> Inventory</button></li>
        <li class="nav-item"><button class="nav-link" id="expenses-tab" data-bs-toggle="pill" data-bs-target="#expenses-content" type="button" onclick="renderStandaloneExpensesLedger()"><i class="fa-solid fa-receipt me-1"></i> Salary & Expenses</button></li>
        <li class="nav-item"><button class="nav-link" id="boss-tab" data-bs-toggle="pill" data-bs-target="#boss-content" type="button" onclick="renderStandaloneBossLedger()"><i class="fa-solid fa-user-tie me-1"></i> D/Eco Boss</button></li>
        <li class="nav-item"><button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateDailyMonthlyAudit()"><i class="fa-solid fa-chart-pie me-1"></i> Audit & Salary Review</button></li>
      </ul>
      <div class="d-flex align-items-center gap-2">
        <button class="btn btn-success btn-sm fw-semibold" onclick="manualSaveData()"><i class="fa-solid fa-floppy-disk me-1"></i> Save Data</button>
        <button class="btn btn-outline-light btn-sm fw-semibold" onclick="location.reload()"><i class="fa-solid fa-rotate me-1"></i> Refresh</button>
        <span class="text-white fw-bold ms-2" id="currentUserName">Administrator</span>
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
            <div><i class="fa-solid fa-info-circle me-1"></i> <strong>Mode:</strong> Regular POS o Inventory Only Mode.</div>
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

      <!-- MANUAL PAYMENT TAB -->
      <div class="tab-pane fade" id="manual-payment-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-2"><i class="fa-solid fa-pen-to-square me-2"></i>Manual Payment & Past Record Entry</h4>
          <p class="text-muted small mb-4">Gamitin ito para i-record ang mga benta noong nakaraang araw na hindi na-encode o para sa karagdagang pondo.</p>
          
          <form id="manualPaymentForm">
            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Petsa ng Transaksyon / Entry:</label>
                <input type="date" id="manualDate" class="form-control" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer / Pinanggalingan:</label>
                <input type="text" id="manualCustomer" class="form-control" placeholder="Hal. Past Sale / Boss" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Uri ng Entry (Type):</label>
                <select id="manualType" class="form-select">
                  <option value="Past Sale (Benta Noon)">Hindi na-encode na Benta (Past Sale)</option>
                  <option value="Dagdag Pera / Capital">Dagdag Pera / Starting Cash / Capital</option>
                  <option value="Manual Adjustment">Manual Adjustment / Iba pa</option>
                </select>
              </div>
            </div>

            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Halaga (Amount ₱):</label>
                <input type="number" step="0.01" id="manualAmount" class="form-control fw-bold text-success fs-5" placeholder="0.00" required>
              </div>
              <div class="col-md-8">
                <label class="form-label fw-semibold">Paliwanag / Remarks:</label>
                <input type="text" id="manualNotes" class="form-control" placeholder="Hal. Nakalimutang i-encode noong Lunes...">
              </div>
            </div>

            <div class="mt-4 text-end">
              <button type="submit" class="btn btn-success btn-lg px-4"><i class="fa-solid fa-floppy-disk me-2"></i>I-save ang Manual Entry</button>
            </div>
          </form>
        </div>
      </div>

      <!-- DATA BACKUP & SAFETY TAB -->
      <div class="tab-pane fade" id="backup-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-shield-halved me-2"></i>Proteksyon sa Data (Backup & Recovery)</h4>
          <p class="text-muted">Para matiyak na hinding-hindi mawawala ang iyong mga transaksyon kahit umabot pa ng sampung taon o mag-palit/masira ang iyong computer, i-download ang backup file buwan-buwan.</p>
          
          <div class="row g-4 mt-2">
            <div class="col-md-6">
              <div class="card bg-light border p-3 h-100">
                <h5 class="fw-bold text-dark mb-2"><i class="fa-solid fa-download text-success me-2"></i>1. I-download ang Backup (Export)</h5>
                <p class="small text-muted">Kumuha ng kopya ng lahat ng iyong data at i-save sa USB o computer.</p>
                <button class="btn btn-success mt-auto fw-bold" onclick="exportDataBackup()"><i class="fa-solid fa-file-arrow-down me-2"></i> I-download ang Buong Backup File (.JSON)</button>
              </div>
            </div>

            <div class="col-md-6">
              <div class="card bg-light border p-3 h-100">
                <h5 class="fw-bold text-dark mb-2"><i class="fa-solid fa-upload text-primary me-2"></i>2. Ibalik ang Data (Restore / Import)</h5>
                <p class="small text-muted">Gamitin ito para ibalik ang mga nakaraang data kung lumipat ka ng ibang computer.</p>
                <input type="file" id="backupFileInput" class="form-control mb-2" accept=".json">
                <button class="btn btn-primary fw-bold" onclick="importDataBackup()"><i class="fa-solid fa-file-arrow-up me-2"></i> I-load / I-restore ang Backup File</button>
              </div>
            </div>
          </div>
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
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;"><span class="text-muted small fw-bold">TOTAL COLLECTION & NET (W/ BOSS)</span><div class="mt-1"><span class="text-success fw-bold" id="dailyTotalCollected">₱0.00</span><br><span class="text-success fw-bold" id="dailyTotalNetProfit">₱0.00</span></div></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;"><span class="text-muted small fw-bold">TOTAL SALES & TX</span><div class="mt-1"><span class="text-primary fw-bold" id="dailyTotalSales">₱0.00</span><br><span class="text-warning fw-bold" id="dailyTxCount">0</span></div></div></div>
          </div>

          <div class="card p-3 bg-light border mb-4">
            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-money-bill-wave me-2"></i>Daily Cash Money Breakdown & Non-Cash Deductions</h6>
            <div class="row g-3">
              <div class="col-md-7">
                <div class="table-responsive">
                  <table class="table table-sm table-bordered bg-white align-middle m-0">
                    <thead class="table-dark"><tr><th>Denomination</th><th style="width: 130px;">Count / Pcs</th><th style="width: 150px;">Subtotal (₱)</th></tr></thead>
                    <tbody>
                      <tr><td class="fw-semibold">₱1,000</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="1000" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">₱500</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="500" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">₱200</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="200" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">₱100</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="100" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">₱50</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="50" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">₱20</td><td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="20" oninput="calculateMoneyBreakdown()"></td><td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td></tr>
                      <tr><td class="fw-semibold">Coins / Barya</td><td><span class="text-muted small">Total Barya</span></td><td><input type="number" step="0.01" min="0" class="form-control form-control-sm denom-coins" placeholder="0.00" oninput="calculateMoneyBreakdown()"></td></tr>
                    </tbody>
                    <tfoot class="table-secondary fw-bold">
                      <tr><td class="text-end">TOTAL COUNTED CASH:</td><td id="breakdownTotalPcs">0 pcs</td><td id="breakdownTotalAmount" class="text-success">₱0.00</td></tr>
                    </tfoot>
                  </table>
                </div>
              </div>
              <div class="col-md-5">
                <div class="card p-3 bg-white h-100 border">
                  <h6 class="fw-bold border-bottom pb-2 mb-2"><i class="fa-solid fa-scale-balanced me-2"></i>Cash Audit & Non-Cash Less</h6>
                  <div class="mb-2 bg-light p-2 rounded">
                    <label class="form-label fw-bold small mb-1">Pondo / Change Fund:</label>
                    <input type="number" step="0.01" min="0" class="form-control form-control-sm denom-fund" id="cashFundInput" placeholder="0.00" oninput="calculateMoneyBreakdown()">
                  </div>
                  <div class="d-flex justify-content-between small mb-1"><span>Total Collections:</span><span id="totalCollectionAll" class="fw-bold">₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Byahe Cash</span><span id="lessByaheCash" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: GCash</span><span id="lessGCash" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Bank Transfer (BT)</span><span id="lessBT" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Cheque</span><span id="lessCheque" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Salary</span><span id="lessSalary" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Expenses</span><span id="lessExpenses" class="text-danger">-₱0.00</span></div>
                  <div class="d-flex justify-content-between small mb-1"><span>Less: Unpaid Balance</span><span id="lessRemainingBalance" class="text-danger">-₱0.00</span></div>
                  <hr class="my-1">
                  <div class="d-flex justify-content-between fw-bold mb-1 bg-light p-1 rounded"><span>Target Cash (Drawer):</span><span id="breakdownTargetWithFund" class="text-primary">₱0.00</span></div>
                  <div class="d-flex justify-content-between fw-bold mb-1"><span>Total Counted Cash:</span><span id="totalCountedCash">₱0.00</span></div>
                  <div class="d-flex justify-content-between fw-bold"><span>Discrepancy (Over/Short):</span><span id="cashDiscrepancy">₱0.00</span></div>
                  <div id="cashStatusAlert" class="alert alert-secondary text-center p-1 small mt-2 fw-bold mb-0">Magpasok ng breakdown.</div>
                </div>
              </div>
            </div>
          </div>

          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr><th>#</th><th>Customer</th><th>Location</th><th>Products</th><th>Container</th><th>Cost</th><th>Total</th><th>Paid</th><th>Balance</th><th>Net Profit</th><th>Payment Method</th><th class="no-print">Action</th></tr>
              </thead>
              <tbody id="dailyTableBody"></tbody>
            </table>
          </div>

          <!-- Hiwalay na Salary at Expenses Section sa Daily Report -->
          <div class="mt-4">
            <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-receipt me-2"></i>Salary & Expenses para sa Petsang Ito</h6>
            <div class="table-responsive">
              <table class="table table-bordered align-middle">
                <thead class="table-dark">
                  <tr>
                    <th>Petsa</th>
                    <th>Paglalarawan (Description)</th>
                    <th style="width: 200px;">Salary (₱)</th>
                    <th style="width: 200px;">Expenses (₱)</th>
                  </tr>
                </thead>
                <tbody id="dailyExpensesTableBody">
                  <tr><td colspan="4" class="text-center text-muted">Walang salary o expenses sa petsang ito.</td></tr>
                </tbody>
                <tfoot class="table-secondary fw-bold">
                  <tr>
                    <td colspan="2" class="text-end">SUBTOTAL / KABUUAN:</td>
                    <td id="dailyTotalSalarySum" class="text-primary">₱0.00</td>
                    <td id="dailyTotalExpensesSum" class="text-danger">₱0.00</td>
                  </tr>
                </tfoot>
              </table>
            </div>
          </div>

          <!-- D/Eco Boss Section sa Daily Report -->
          <div class="mt-4">
            <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss para sa Petsang Ito (Idinaragdag sa Net Profit)</h6>
            <div class="table-responsive">
              <table class="table table-bordered align-middle">
                <thead class="table-dark">
                  <tr>
                    <th>Petsa</th>
                    <th>Detalye / Remarks</th>
                    <th style="width: 200px;">Halaga (₱)</th>
                  </tr>
                </thead>
                <tbody id="dailyBossTableBody">
                  <tr><td colspan="3" class="text-center text-muted">Walang record para kay Boss sa petsang ito.</td></tr>
                </tbody>
                <tfoot class="table-secondary fw-bold">
                  <tr>
                    <td colspan="2" class="text-end">TOTAL D/ECO BOSS:</td>
                    <td id="dailyTotalBossSum" class="text-success">₱0.00</td>
                  </tr>
                </tfoot>
              </table>
            </div>
          </div>

        </div>
      </div>

      <!-- 3. UTANG & PAYMENTS TAB -->
      <div class="tab-pane fade" id="credit-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-users-viewfinder me-2"></i>Utang & Payments</h4>
          <div class="table-responsive">
            <table class="table table-hover align-middle">
              <thead class="table-dark"><tr><th>Customer</th><th>Location</th><th>Product</th><th>Total (₱)</th><th>Paid (₱)</th><th>Balance (₱)</th><th>Due Date</th><th>Status</th><th class="no-print" style="width: 120px;">Action</th></tr></thead>
              <tbody id="creditTableBody"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 3.5 ORDER LOOKUP & LAST PRICE REVIEW TAB -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-magnifying-glass me-2"></i>Customer Last Order & Price Review</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()"><i class="fa-solid fa-print me-1"></i> Print Record</button>
          </div>
          
          <div class="row g-2 mb-4">
            <div class="col-md-9"><input type="text" id="searchCustomerInput" class="form-control form-control-lg" placeholder="I-type ang pangalan ng Customer (e.g., Juan Dela Cruz)..."></div>
            <div class="col-md-3"><button class="btn btn-primary btn-lg w-100 fw-bold" onclick="searchCustomerOrder()"><i class="fa-solid fa-search me-1"></i> Search Order</button></div>
          </div>

          <div id="searchResultContainer" style="display: none;">
            <div class="card bg-light border-primary mb-4">
              <div class="card-header bg-primary text-white fw-bold d-flex justify-content-between align-items-center">
                <span><i class="fa-solid fa-receipt me-2"></i>Detalye ng Huling Order (Last Order Details & Price)</span>
                <span id="lastOrderBadge" class="badge bg-warning text-dark fs-6">Status</span>
              </div>
              <div class="card-body">
                <div class="row g-3">
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">CUSTOMER NAME:</p><h5 class="fw-bold text-dark" id="lastOrderCustomer">-</h5></div>
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">DATE OF PURCHASE:</p><h5 class="fw-bold text-dark" id="lastOrderDate">-</h5></div>
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">LOCATION:</p><h5 class="fw-bold text-dark" id="lastOrderLocation">-</h5></div>
                  <div class="col-md-12"><p class="mb-1 text-muted small fw-bold">ITEMS / PRODUCTS & PRICES:</p><div class="table-responsive"><table class="table table-sm table-bordered bg-white mb-0"><thead class="table-light"><tr><th>Product Name</th><th>Qty</th><th>Unit Price (₱)</th><th>Subtotal (₱)</th></tr></thead><tbody id="lastOrderItemsBody"></tbody></table></div></div>
                  <hr class="my-2">
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">TOTAL AMOUNT:</p><h4 class="fw-bold text-primary" id="lastOrderTotal">₱0.00</h4></div>
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">PAID AMOUNT:</p><h4 class="fw-bold text-success" id="lastOrderPaid">₱0.00</h4></div>
                  <div class="col-md-4"><p class="mb-1 text-muted small fw-bold">REMAINING BALANCE:</p><h4 class="fw-bold text-danger" id="lastOrderBalance">₱0.00</h4></div>
                </div>
              </div>
            </div>

            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-clock-rotate-left me-2"></i>Buong Kasaysayan ng Transaksyon ni Customer (Complete History)</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark">
                  <tr><th>Date</th><th>Location</th><th>Products Summary</th><th>Total (₱)</th><th>Paid (₱)</th><th>Balance (₱)</th><th>Status</th></tr>
                </thead>
                <tbody id="customerHistoryBody"></tbody>
              </table>
            </div>
          </div>

          <div id="noCustomerFound" class="alert alert-warning text-center p-3 d-none">
            <i class="fa-solid fa-triangle-exclamation me-2"></i> Walang nahanap na record para sa customer na ito.
          </div>
        </div>
      </div>

      <!-- 4. INVENTORY TAB -->
      <div class="tab-pane fade" id="inventory-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-boxes-stacked me-2"></i>Advanced Inventory Management</h4>
          <p class="text-muted small">I-setup minsan ang mga produkto sa <strong>Master Product List</strong>. Pagkatapos, maaari ka nang mag-log ng Araw-araw na Pumasok, Return, at mag-review per day.</p>

          <!-- Select Date for Inventory View -->
          <div class="row g-3 align-items-center mb-4 bg-light p-3 rounded border">
            <div class="col-md-4">
              <label class="form-label fw-bold mb-0">Piliin ang Petsa ng Inventory (Date):</label>
            </div>
            <div class="col-md-4">
              <input type="date" id="inventoryViewDate" class="form-control" onchange="renderInventoryTables()">
            </div>
          </div>
          
          <!-- 1. Master Product List Form (Permanent Column) -->
          <div class="card bg-white p-3 mb-4 border shadow-sm">
            <h6 class="fw-bold text-primary mb-2"><i class="fa-solid fa-list-check me-1"></i> Master Product Setup (Permanenteng Listahan ng mga Produkto)</h6>
            <p class="text-muted small mb-2">Dito mo lang i-a-add minsan ang pangalan ng item, manual description, at supplier.</p>
            <div class="row g-2">
              <div class="col-md-4">
                <input type="text" id="masterProductName" class="form-control form-control-sm" placeholder="Pangalan ng Produkto (Hal. Palm Oil)">
              </div>
              <div class="col-md-3">
                <input type="text" id="masterDescription" class="form-control form-control-sm" placeholder="Description / Uri (Hal. 1 Liter)">
              </div>
              <div class="col-md-3">
                <input type="text" id="masterSupplier" class="form-control form-control-sm" placeholder="Supplier (Hal. ABC Supplier)">
              </div>
              <div class="col-md-2">
                <button class="btn btn-success btn-sm w-100 fw-bold" onclick="addMasterProduct()"><i class="fa-solid fa-plus me-1"></i> I-save sa Masterlist</button>
              </div>
            </div>
          </div>

          <!-- Daily Stock In & Return Form -->
          <div class="card bg-light p-3 mb-4 border">
            <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-pen-to-square me-1"></i> Mag-log ng Pumasok na Stock o Return (Para sa Napiling Petsa sa Itaas)</h6>
            <div class="row g-2">
              <div class="col-md-4">
                <select id="logProductSelect" class="form-select form-select-sm">
                  <option value="">-- Piliin ang Produkto mula sa Masterlist --</option>
                </select>
              </div>
              <div class="col-md-2">
                <input type="number" step="any" id="logQtyIn" class="form-control form-control-sm" placeholder="Pumasok (In)" value="0">
              </div>
              <div class="col-md-2">
                <input type="number" step="any" id="logReturn" class="form-control form-control-sm" placeholder="Return / Sauli" value="0">
              </div>
              <div class="col-md-4">
                <button class="btn btn-primary btn-sm w-100 fw-bold" onclick="saveDailyInventoryLog()">I-save ang Araw-araw na Galaw</button>
              </div>
            </div>
          </div>

          <h5 class="fw-bold text-secondary mt-3">1. PALM & COCO INVENTORY (Per Day / Ending & Beginning)</h5>
          <div class="table-responsive mb-4">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr><th>Produkto</th><th>Description</th><th>Supplier</th><th>Beginning</th><th>Pumasok (In)</th><th>Return/Sauli</th><th>Nabenta (Sold)</th><th>Ending Stock</th></tr>
              </thead>
              <tbody id="palmCocoInventoryBody">
                <tr><td colspan="8" class="text-center text-muted">Walang record.</td></tr>
              </tbody>
            </table>
          </div>

          <h5 class="fw-bold text-secondary mt-3">2. IBA PANG ITEMS INVENTORY (Per Day / Ending & Beginning)</h5>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr><th>Produkto</th><th>Description</th><th>Supplier</th><th>Beginning</th><th>Pumasok (In)</th><th>Return/Sauli</th><th>Nabenta (Sold)</th><th>Ending Stock</th></tr>
              </thead>
              <tbody id="otherInventoryBody">
                <tr><td colspan="8" class="text-center text-muted">Walang record.</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 5. EXPENSES TAB -->
      <div class="tab-pane fade" id="expenses-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-receipt me-2"></i>Salary & Expenses Ledger (Per Day)</h4>
          <p class="text-muted small">Mag-record ng mga araw-araw na gastos o sweldo. May hiwalay na column para sa Salary at Expenses para mas madaling ma-subtotal.</p>
          
          <div class="row g-3 mb-3">
            <div class="col-md-2">
              <label class="form-label fw-semibold">Petsa:</label>
              <input type="date" id="expenseDate" class="form-control">
            </div>
            <div class="col-md-3">
              <label class="form-label fw-semibold">Uri (Type):</label>
              <select id="expenseType" class="form-select">
                <option value="Salary">Salary (Sweldo)</option>
                <option value="Expense">Expense (Gastos)</option>
              </select>
            </div>
            <div class="col-md-4">
              <label class="form-label fw-semibold">Paglalarawan / Title:</label>
              <input type="text" id="expenseTitle" class="form-control" placeholder="Hal. Sweldo ni Juan / Kuryente">
            </div>
            <div class="col-md-3">
              <label class="form-label fw-semibold">Halaga (₱):</label>
              <input type="number" step="0.01" id="expenseAmount" class="form-control" placeholder="0.00">
            </div>
          </div>
          <div class="text-end mb-4">
            <button class="btn btn-primary px-4 fw-bold" onclick="addExpenseItem()"><i class="fa-solid fa-plus me-1"></i> Idagdag sa Talaan</button>
          </div>

          <div class="table-responsive">
            <table class="table table-bordered align-middle">
              <thead class="table-dark">
                <tr>
                  <th>Petsa</th>
                  <th>Paglalarawan (Description)</th>
                  <th style="width: 200px;">Salary (₱)</th>
                  <th style="width: 200px;">Expenses (₱)</th>
                  <th class="no-print" style="width: 80px;">Aksyon</th>
                </tr>
              </thead>
              <tbody id="expensesTableBody">
                <tr><td colspan="5" class="text-center text-muted">Walang record na naitala.</td></tr>
              </tbody>
              <tfoot class="table-secondary fw-bold">
                <tr>
                  <td colspan="2" class="text-end">SUBTOTAL / KABUUAN:</td>
                  <td id="totalSalarySum" class="text-primary">₱0.00</td>
                  <td id="totalExpensesSum" class="text-danger">₱0.00</td>
                  <td></td>
                </tr>
              </tfoot>
            </table>
          </div>
        </div>
      </div>

      <!-- 6. BOSS TAB -->
      <div class="tab-pane fade" id="boss-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-3"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Ledger</h4>
          <p class="text-muted small">Talaan ng mga transaksyon, bigayan, o hulugan para kay Boss.</p>
          <div class="row g-3 mb-3">
            <div class="col-md-3">
              <label class="form-label fw-semibold">Petsa:</label>
              <input type="date" id="bossDate" class="form-control">
            </div>
            <div class="col-md-4">
              <label class="form-label fw-semibold">Detalye / Remarks:</label>
              <input type="text" id="bossTitle" class="form-control" placeholder="Hal. Bigay kay Boss / Remittance">
            </div>
            <div class="col-md-3">
              <label class="form-label fw-semibold">Halaga (₱):</label>
              <input type="number" step="0.01" id="bossAmount" class="form-control" placeholder="0.00">
            </div>
            <div class="col-md-2 d-flex align-items-end">
              <button class="btn btn-success w-100 fw-bold" onclick="addBossItem()">I-record</button>
            </div>
          </div>
          <div class="table-responsive">
            <table class="table table-bordered align-middle">
              <thead class="table-dark"><tr><th>Petsa</th><th>Detalye</th><th>Halaga (₱)</th><th class="no-print">Aksyon</th></tr></thead>
              <tbody id="bossTableBody">
                <tr><td colspan="4" class="text-center text-muted">Walang record para kay Boss.</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 7. AUDIT TAB (MONTHLY & DAILY AUDIT REVIEW) -->
      <div class="tab-pane fade" id="audit-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-chart-pie me-2"></i>Daily & Monthly Audit Review</h4>
            <div class="d-flex gap-2 align-items-center">
              <label class="fw-bold me-1">Piliin ang Petsa/Buwan:</label>
              <input type="date" id="auditDateInput" class="form-control" onchange="generateDailyMonthlyAudit()">
            </div>
          </div>

          <!-- ARAW NA ITO (Daily Summary Cards) -->
          <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-calendar-day me-2"></i>Pangkalahatan para sa Araw na Ito</h5>
          <div class="row g-3 mb-4">
            <div class="col-md-3"><div class="card p-3 stat-card bg-light"><span class="text-muted small fw-bold">BENTA SA ARAW NA ITO</span><h4 class="text-primary fw-bold mt-1" id="auditDaySales">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;"><span class="text-muted small fw-bold">NET PROFIT SA ARAW NA ITO</span><h4 class="text-success fw-bold mt-1" id="auditDayProfit">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;"><span class="text-muted small fw-bold">GASTOS & SWELDO SA ARAW</span><h4 class="text-danger fw-bold mt-1" id="auditDayExpenses">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #8e44ad;"><span class="text-muted small fw-bold">NET KITA (WITH BOSS)</span><h4 class="text-dark fw-bold mt-1" id="auditDayNetIncome">₱0.00</h4></div></div>
          </div>

          <!-- KABUUANG BUWAN (Monthly Summary Cards) -->
          <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-calendar-days me-2"></i>Kabuuang Buwan (Monthly Total Summary)</h5>
          <div class="row g-3 mb-4">
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #0288d1;"><span class="text-muted small fw-bold">KABUUANG BENTA (BUWAN)</span><h4 class="text-primary fw-bold mt-1" id="auditMonthSales">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #388e3c;"><span class="text-muted small fw-bold">KABUUANG NET PROFIT (BUWAN)</span><h4 class="text-success fw-bold mt-1" id="auditMonthProfit">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #e65100;"><span class="text-muted small fw-bold">KABUUANG GASTOS & SWELDO (BUWAN)</span><h4 class="text-danger fw-bold mt-1" id="auditMonthExpenses">₱0.00</h4></div></div>
            <div class="col-md-3"><div class="card p-3 stat-card bg-light" style="border-left-color: #7b1fa2;"><span class="text-muted small fw-bold">KABUUANG NET KITA (WITH BOSS - BUWAN)</span><h4 class="text-dark fw-bold mt-1" id="auditMonthNetIncome">₱0.00</h4></div></div>
          </div>

          <h5 class="fw-bold text-secondary mb-3">Detalyadong Listahan ng Transaksyon at Expenses sa Napiling Araw</h5>
          <div class="table-responsive mb-4">
            <table class="table table-bordered align-middle">
              <thead class="table-dark">
                <tr>
                  <th>Uri</th>
                  <th>Pangalan / Customer / Paglalarawan</th>
                  <th>Location / Kategoriya</th>
                  <th style="width: 150px;">Salary (₱)</th>
                  <th style="width: 150px;">Expenses (₱)</th>
                  <th style="width: 150px;">D/Eco Boss (₱)</th>
                  <th style="width: 150px;">Net Profit (₱)</th>
                </tr>
              </thead>
              <tbody id="auditDailyDetailBody">
                <tr><td colspan="7" class="text-center text-muted">Walang record sa petsang ito.</td></tr>
              </tbody>
            </table>
          </div>

          <!-- Detalyadong Listahan para sa Buong Buwan -->
          <h5 class="fw-bold text-secondary mb-3">Kabuuang Listahan ng Buong Buwan (Monthly Consolidated List)</h5>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-secondary">
                <tr>
                  <th>Petsa</th>
                  <th>Uri</th>
                  <th>Pangalan / Customer / Paglalarawan</th>
                  <th>Location</th>
                  <th style="width: 130px;">Salary (₱)</th>
                  <th style="width: 130px;">Expenses (₱)</th>
                  <th style="width: 130px;">D/Eco Boss (₱)</th>
                  <th style="width: 130px;">Net Profit (₱)</th>
                </tr>
              </thead>
              <tbody id="auditMonthlyDetailBody">
                <tr><td colspan="8" class="text-center text-muted">Walang record sa buwang ito.</td></tr>
              </tbody>
              <tfoot class="table-dark fw-bold">
                <tr>
                  <td colspan="4" class="text-end">KABUUANG BUUAN (MONTHLY TOTAL):</td>
                  <td id="auditMonthTotalSal">₱0.00</td>
                  <td id="auditMonthTotalExp">₱0.00</td>
                  <td id="auditMonthTotalBoss">₱0.00</td>
                  <td id="auditMonthTotalProfit">₱0.00</td>
                </tr>
              </tfoot>
            </table>
          </div>

        </div>
      </div>

    </div>
  </div>

  <!-- EDIT CREDIT MODAL -->
  <div class="modal fade" id="editCreditModal" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-pen-to-square me-2"></i>I-edit ang Utang / Transaksyon</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <input type="hidden" id="editCreditId">
          <div class="mb-3">
            <label class="form-label fw-semibold">Customer Name:</label>
            <input type="text" id="editCustomerName" class="form-control" required>
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Total Amount (₱):</label>
            <input type="number" step="0.01" id="editTotalAmount" class="form-control" required oninput="calculateEditBalance()">
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Binayad na Halaga (Paid Amount ₱):</label>
            <input type="number" step="0.01" id="editPaidAmount" class="form-control" required oninput="calculateEditBalance()">
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Remaining Balance (₱):</label>
            <input type="number" step="0.01" id="editRemainingBalance" class="form-control bg-light" readonly>
          </div>
          <div class="mb-3">
            <label class="form-label fw-semibold">Due Date:</label>
            <input type="date" id="editDueDate" class="form-control">
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Isara</button>
          <button type="button" class="btn btn-primary fw-bold" onclick="saveEditedCredit()">I-save ang Pagbabago</button>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let inventoryMasterList = JSON.parse(localStorage.getItem('rmv_inventoryMasterList')) || [];
    let inventoryDailyLogs = JSON.parse(localStorage.getItem('rmv_inventoryDailyLogs')) || {};
    let cashBreakdownData = JSON.parse(localStorage.getItem('rmv_cashBreakdownData')) || {};
    let monthlyExpensesData = JSON.parse(localStorage.getItem('rmv_monthlyExpensesData')) || {};
    let bossLedgerData = JSON.parse(localStorage.getItem('rmv_bossLedgerData')) || {};

    function getTodayDateString() {
      const now = new Date();
      return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;
    }

    let todayFormatted = getTodayDateString();
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;
    document.getElementById('manualDate').value = todayFormatted;
    document.getElementById('expenseDate').value = todayFormatted;
    document.getElementById('bossDate').value = todayFormatted;
    document.getElementById('inventoryViewDate').value = todayFormatted;
    document.getElementById('auditDateInput').value = todayFormatted;

    window.onload = function() {
      addPosRow();
      loadMoneyBreakdown();
      generateDailyReport();
      renderCreditTable();
      renderInventoryTables();
      renderStandaloneExpensesLedger();
      renderStandaloneBossLedger();
      generateDailyMonthlyAudit();
    };

    function saveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_inventoryMasterList', JSON.stringify(inventoryMasterList));
      localStorage.setItem('rmv_inventoryDailyLogs', JSON.stringify(inventoryDailyLogs));
      localStorage.setItem('rmv_cashBreakdownData', JSON.stringify(cashBreakdownData));
      localStorage.setItem('rmv_monthlyExpensesData', JSON.stringify(monthlyExpensesData));
      localStorage.setItem('rmv_bossLedgerData', JSON.stringify(bossLedgerData));
    }

    function manualSaveData() {
      saveData();
      alert('Data saved successfully!');
    }

    function exportDataBackup() {
      const backupData = {
        version: "5.5",
        exportDate: new Date().toISOString(),
        transactions, inventoryMasterList, inventoryDailyLogs, cashBreakdownData, monthlyExpensesData, bossLedgerData
      };
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(backupData, null, 2));
      const downloadAnchor = document.createElement('a');
      downloadAnchor.setAttribute("href", dataStr);
      downloadAnchor.setAttribute("download", `RMVillasis_POS_Backup_${getTodayDateString()}.json`);
      document.body.appendChild(downloadAnchor);
      downloadAnchor.click();
      downloadAnchor.remove();
      alert('Tagumpay na na-download ang iyong backup file!');
    }

    function importDataBackup() {
      const fileInput = document.getElementById('backupFileInput');
      if (fileInput.files.length === 0) {
        alert('Mangyaring pumili muna ng backup (.json) file.');
        return;
      }
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        try {
          const imported = JSON.parse(e.target.result);
          if (confirm('Sigurado ka bang gusto mong i-restore ang data na ito? Mapapalitan nito ang kasalukuyang data.')) {
            if (imported.transactions) transactions = imported.transactions;
            if (imported.inventoryMasterList) inventoryMasterList = imported.inventoryMasterList;
            if (imported.inventoryDailyLogs) inventoryDailyLogs = imported.inventoryDailyLogs;
            if (imported.cashBreakdownData) cashBreakdownData = imported.cashBreakdownData;
            if (imported.monthlyExpensesData) monthlyExpensesData = imported.monthlyExpensesData;
            if (imported.bossLedgerData) bossLedgerData = imported.bossLedgerData;
            saveData();
            alert('Tagumpay na nai-restore ang lahat ng data!');
            location.reload();
          }
        } catch (error) {
          alert('May error sa file na iyong binuksan.');
        }
      };
      reader.readAsText(file);
    }

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
          const mCash = parseFloat(document.getElementById('multiCashAmt').value) || 0;
          const mGcash = parseFloat(document.getElementById('multiGcashAmt').value) || 0;
          const mBT = parseFloat(document.getElementById('multiBTAmt').value) || 0;
          const mOther = parseFloat(document.getElementById('multiOtherAmt').value) || 0;
          if (mCash > 0) payments.push({ amount: mCash, method: 'Cash', date });
          if (mGcash > 0) payments.push({ amount: mGcash, method: 'GCash', date });
          if (mBT > 0) payments.push({ amount: mBT, method: 'Bank Transfer', date });
          if (mOther > 0) payments.push({ amount: mOther, method: 'Byahe Cash', date });
        } else if (paid > 0) {
          payments.push({ amount: paid, method, date });
        }
      }

      let itemsList = [], productSummary = [], totalCost = 0;
      document.querySelectorAll('#posItemsBody tr').forEach(row => {
        const name = row.querySelector('.pos-name').value.trim();
        const desc = row.querySelector('.pos-desc').value.trim();
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = isInv ? 0 : (parseFloat(row.querySelector('.pos-cost').value) || 0);
        const price = isInv ? 0 : (parseFloat(row.querySelector('.pos-price').value) || 0);
        totalCost += (qty * cost);
        if (name) {
          productSummary.push(`${name} (x${qty})`);
          itemsList.push({ name, desc, qty, cost, price, subtotal: qty * price });
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
      renderInventoryTables();
    });

    document.getElementById('manualPaymentForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('manualDate').value;
      const cust = document.getElementById('manualCustomer').value.trim();
      const type = document.getElementById('manualType').value;
      const amount = parseFloat(document.getElementById('manualAmount').value) || 0;
      const notes = document.getElementById('manualNotes').value.trim();

      transactions.push({
        id: Date.now(),
        date: date,
        customer: cust,
        location: 'Hiway',
        product: `[${type}] ${notes}`,
        itemsList: [],
        total: amount,
        totalCost: 0,
        paid: amount,
        balance: 0,
        status: 'PAID',
        payments: [{ amount: amount, method: 'Cash', date: date }]
      });

      saveData();
      alert('Tagumpay na naidagdag ang Manual Payment / Past Entry!');
      this.reset();
      document.getElementById('manualDate').value = getTodayDateString();
      generateDailyReport();
    });

    function generateDailyReport() {
      const date = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';
      let totalSales = 0, totalHiwaySales = 0, totalByaheSales = 0, totalHiwayProfit = 0, totalByaheProfit = 0;
      let totalCollected = 0, totalNet = 0, count = 0;
      let nonCashByaheCash = 0, nonCashGCash = 0, nonCashBT = 0, nonCashCheque = 0;
      let dayRemainingBalance = 0;

      transactions.forEach((t, index) => {
        const txCost = t.totalCost || 0;
        const netProf = t.total - txCost;

        if (t.date === date) {
          totalSales += t.total;
          totalNet += netProf;
          dayRemainingBalance += (t.balance || 0);
          if (t.location === 'Hiway') { totalHiwaySales += t.total; totalHiwayProfit += netProf; }
          else if (t.location === 'Byahe') { totalByaheSales += t.total; totalByaheProfit += netProf; }
        }

        if (t.payments) {
          t.payments.forEach(p => {
            if (p.date === date) {
              totalCollected += p.amount;
              const mLower = p.method.toLowerCase();
              if (mLower.includes('byahe')) {
                nonCashByaheCash += p.amount;
              } else if (mLower.includes('gcash')) {
                nonCashGCash += p.amount;
              } else if (mLower.includes('bank') || mLower.includes('bt')) {
                nonCashBT += p.amount;
              } else if (mLower.includes('cheque')) {
                nonCashCheque += p.amount;
              }
            }
          });
        }
        count++;

        let methodStr = t.payments ? t.payments.map(p => `${p.method}: ₱${p.amount.toFixed(2)}`).join(', ') : 'Cash';

        tbody.innerHTML += `
          <tr>
            <td>${index+1}</td>
            <td class="fw-bold">${t.customer}</td>
            <td><span class="badge bg-secondary">${t.location}</span></td>
            <td>${t.product}</td>
            <td>${t.containerInfo || 'Wala'}</td>
            <td>₱${txCost.toFixed(2)}</td>
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
      });

      if(count === 0) {
        tbody.innerHTML = `<tr><td colspan="12" class="text-center text-muted py-3">Wala pang transaksyon sa araw na ito.</td></tr>`;
      }

      // Render Daily Expenses & Salary Table
      const expTbody = document.getElementById('dailyExpensesTableBody');
      expTbody.innerHTML = '';
      let daySalarySum = 0;
      let dayExpensesSum = 0;
      let expCount = 0;

      if (monthlyExpensesData[date]) {
        monthlyExpensesData[date].forEach(ex => {
          expCount++;
          let isSalary = (ex.type === 'Salary' || (!ex.type && ex.title.toLowerCase().includes('sweldo')));
          let salVal = isSalary ? ex.amount : 0;
          let expVal = !isSalary ? ex.amount : 0;

          daySalarySum += salVal;
          dayExpensesSum += expVal;

          expTbody.innerHTML += `
            <tr>
              <td>${date}</td>
              <td class="fw-bold">${ex.title}</td>
              <td class="text-primary">${salVal > 0 ? '₱' + salVal.toFixed(2) : '-'}</td>
              <td class="text-danger">${expVal > 0 ? '₱' + expVal.toFixed(2) : '-'}</td>
            </tr>
          `;
        });
      }

      if (expCount === 0) {
        expTbody.innerHTML = `<tr><td colspan="4" class="text-center text-muted">Walang salary o expenses sa petsang ito.</td></tr>`;
      }

      document.getElementById('dailyTotalSalarySum').innerText = `₱${daySalarySum.toFixed(2)}`;
      document.getElementById('dailyTotalExpensesSum').innerText = `₱${dayExpensesSum.toFixed(2)}`;

      // Render Daily Boss Table & Sum
      const bossTbody = document.getElementById('dailyBossTableBody');
      bossTbody.innerHTML = '';
      let dayBossSum = 0;
      let bossCount = 0;

      if (bossLedgerData[date]) {
        bossLedgerData[date].forEach(b => {
          bossCount++;
          dayBossSum += (b.amount || 0);
          bossTbody.innerHTML += `
            <tr>
              <td>${date}</td>
              <td class="fw-bold">${b.title}</td>
              <td class="text-success">₱${b.amount.toFixed(2)}</td>
            </tr>
          `;
        });
      }

      if (bossCount === 0) {
        bossTbody.innerHTML = `<tr><td colspan="3" class="text-center text-muted">Walang record para kay Boss sa petsang ito.</td></tr>`;
      }
      document.getElementById('dailyTotalBossSum').innerText = `₱${dayBossSum.toFixed(2)}`;

      document.getElementById('dailyHiwaySales').innerText = `₱${totalHiwaySales.toFixed(2)}`;
      document.getElementById('dailyHiwayProfit').innerText = `₱${totalHiwayProfit.toFixed(2)}`;
      document.getElementById('dailyByaheSales').innerText = `₱${totalByaheSales.toFixed(2)}`;
      document.getElementById('dailyByaheProfit').innerText = `₱${totalByaheProfit.toFixed(2)}`;
      document.getElementById('dailyTotalSales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${totalCollected.toFixed(2)}`;
      
      // Total Net Profit kasama ang D/Eco Boss
      const finalNetWithBoss = totalNet + dayBossSum;
      document.getElementById('dailyTotalNetProfit').innerText = `₱${finalNetWithBoss.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;

      window.currentDayNonCash = { byaheCash: nonCashByaheCash, gcash: nonCashGCash, bt: nonCashBT, cheque: nonCashCheque, remainingBalance: dayRemainingBalance, totalCollected, salarySum: daySalarySum, expensesSum: dayExpensesSum };
      calculateMoneyBreakdown();
    }

    document.querySelectorAll('.denom-count').forEach(input => {
      input.addEventListener('input', calculateMoneyBreakdown);
    });

    function calculateMoneyBreakdown() {
      const date = document.getElementById('dailyReportDate').value;
      let totalPcs = 0, totalCash = 0;
      
      document.querySelectorAll('.denom-count').forEach(input => {
        const denom = parseFloat(input.getAttribute('data-denom')) || 0;
        const count = parseFloat(input.value) || 0;
        const sub = denom * count;
        totalPcs += count;
        totalCash += sub;
        input.closest('tr').querySelector('.denom-subtotal').value = sub.toFixed(2);
      });

      const coins = parseFloat(document.querySelector('.denom-coins').value) || 0;
      totalCash += coins;

      document.getElementById('breakdownTotalPcs').innerText = `${totalPcs} pcs`;
      document.getElementById('breakdownTotalAmount').innerText = `₱${totalCash.toFixed(2)}`;

      const fund = parseFloat(document.getElementById('cashFundInput').value) || 0;
      const nonCash = window.currentDayNonCash || { byaheCash: 0, gcash: 0, bt: 0, cheque: 0, remainingBalance: 0, totalCollected: 0, salarySum: 0, expensesSum: 0 };
      
      document.getElementById('totalCollectionAll').innerText = `₱${nonCash.totalCollected.toFixed(2)}`;
      document.getElementById('lessByaheCash').innerText = `-₱${nonCash.byaheCash.toFixed(2)}`;
      document.getElementById('lessGCash').innerText = `-₱${nonCash.gcash.toFixed(2)}`;
      document.getElementById('lessBT').innerText = `-₱${nonCash.bt.toFixed(2)}`;
      document.getElementById('lessCheque').innerText = `-₱${nonCash.cheque.toFixed(2)}`;
      document.getElementById('lessSalary').innerText = `-₱${nonCash.salarySum.toFixed(2)}`;
      document.getElementById('lessExpenses').innerText = `-₱${nonCash.expensesSum.toFixed(2)}`;
      document.getElementById('lessRemainingBalance').innerText = `-₱${nonCash.remainingBalance.toFixed(2)}`;

      const targetCash = nonCash.totalCollected - nonCash.byaheCash - nonCash.gcash - nonCash.bt - nonCash.cheque - nonCash.salarySum - nonCash.expensesSum - nonCash.remainingBalance;
      const targetWithFund = targetCash + fund;

      document.getElementById('breakdownTargetWithFund').innerText = `₱${targetWithFund.toFixed(2)}`;
      document.getElementById('totalCountedCash').innerText = `₱${totalCash.toFixed(2)}`;

      const discrepancy = totalCash - targetWithFund;
      const discEl = document.getElementById('cashDiscrepancy');
      discEl.innerText = `₱${discrepancy.toFixed(2)}`;
      discEl.className = discrepancy === 0 ? 'fs-5 fw-bold text-success' : (discrepancy > 0 ? 'fs-5 fw-bold text-primary' : 'fs-5 fw-bold text-danger');

      if (!cashBreakdownData[date]) cashBreakdownData[date] = {};
      cashBreakdownData[date].fund = fund;
      cashBreakdownData[date].coins = coins;
      cashBreakdownData[date].counts = {};
      document.querySelectorAll('.denom-count').forEach(input => {
        cashBreakdownData[date].counts[input.getAttribute('data-denom')] = parseFloat(input.value) || 0;
      });
      saveData();
    }

    function loadMoneyBreakdown() {
      const date = document.getElementById('dailyReportDate').value;
      const data = cashBreakdownData[date];
      if (data) {
        document.getElementById('cashFundInput').value = data.fund || 0;
        document.querySelector('.denom-coins').value = data.coins || 0;
        if (data.counts) {
          document.querySelectorAll('.denom-count').forEach(input => {
            input.value = data.counts[input.getAttribute('data-denom')] || 0;
          });
        }
      } else {
        document.getElementById('cashFundInput').value = 0;
        document.querySelector('.denom-coins').value = 0;
        document.querySelectorAll('.denom-count').forEach(input => input.value = 0);
      }
    }

    function renderCreditTable() {
      const tbody = document.getElementById('creditTableBody');
      tbody.innerHTML = '';
      let count = 0;
      transactions.forEach(t => {
        if (t.balance > 0.01) {
          count++;
          tbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td><span class="badge bg-secondary">${t.location}</span></td>
              <td>${t.product}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
              <td>${t.dueDate || 'N/A'}</td>
              <td><span class="badge ${t.status === 'PAID' ? 'bg-success' : 'bg-warning text-dark'}">${t.status}</span></td>
              <td class="no-print text-center">
                <button class="btn btn-sm btn-outline-primary border-0 p-1 me-1" onclick="openEditCreditModal(${t.id})" title="I-edit ang Utang"><i class="fa-solid fa-pen-to-square"></i></button>
                <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteTransaction(${t.id})" title="Burahin"><i class="fa-solid fa-trash-can"></i></button>
              </td>
            </tr>
          `;
        }
      });
      if (count === 0) {
        tbody.innerHTML = `<tr><td colspan="9" class="text-center text-muted py-3">Walang kasalukuyang utang o balanse.</td></tr>`;
      }
    }

    function openEditCreditModal(id) {
      const t = transactions.find(item => item.id === id);
      if (!t) return;
      document.getElementById('editCreditId').value = t.id;
      document.getElementById('editCustomerName').value = t.customer;
      document.getElementById('editTotalAmount').value = t.total;
      document.getElementById('editPaidAmount').value = t.paid;
      document.getElementById('editRemainingBalance').value = t.balance.toFixed(2);
      document.getElementById('editDueDate').value = t.dueDate || '';

      const editModal = new bootstrap.Modal(document.getElementById('editCreditModal'));
      editModal.show();
    }

    function calculateEditBalance() {
      const total = parseFloat(document.getElementById('editTotalAmount').value) || 0;
      const paid = parseFloat(document.getElementById('editPaidAmount').value) || 0;
      const balance = Math.max(0, total - paid);
      document.getElementById('editRemainingBalance').value = balance.toFixed(2);
    }

    function saveEditedCredit() {
      const id = parseInt(document.getElementById('editCreditId').value);
      const cust = document.getElementById('editCustomerName').value.trim();
      const total = parseFloat(document.getElementById('editTotalAmount').value) || 0;
      const paid = parseFloat(document.getElementById('editPaidAmount').value) || 0;
      const balance = Math.max(0, total - paid);
      const dueDate = document.getElementById('editDueDate').value;

      const t = transactions.find(item => item.id === id);
      if (t) {
        t.customer = cust;
        t.total = total;
        t.paid = paid;
        t.balance = balance;
        t.dueDate = dueDate;
        t.status = balance > 0 && paid > 0 ? "PARTIAL" : (balance > 0 ? "UNPAID" : "PAID");

        saveData();
        renderCreditTable();
        generateDailyReport();

        const modalEl = document.getElementById('editCreditModal');
        const modalInstance = bootstrap.Modal.getInstance(modalEl);
        modalInstance.hide();
        alert('Matagumpay na na-update ang utang!');
      }
    }

    // --- MASTER PRODUCT LIST & DAILY INVENTORY MANAGEMENT ---
    function addMasterProduct() {
      const name = document.getElementById('masterProductName').value.trim();
      const description = document.getElementById('masterDescription').value.trim();
      const supplier = document.getElementById('masterSupplier').value.trim();

      if (!name) {
        alert('Mangyaring ilagay ang pangalan ng produkto.');
        return;
      }

      if (inventoryMasterList.some(p => p.name.toLowerCase() === name.toLowerCase())) {
        alert('Ang produktong ito ay nasa masterlist na.');
        return;
      }

      inventoryMasterList.push({ id: Date.now(), name, description: description || 'N/A', supplier: supplier || 'N/A' });
      saveData();
      renderInventoryTables();
      document.getElementById('masterProductName').value = '';
      document.getElementById('masterDescription').value = '';
      document.getElementById('masterSupplier').value = '';
      alert('Matagumpay na naidagdag sa Masterlist!');
    }

    function saveDailyInventoryLog() {
      const date = document.getElementById('inventoryViewDate').value;
      const select = document.getElementById('logProductSelect');
      const prodName = select.value;
      const qtyIn = parseFloat(document.getElementById('logQtyIn').value) || 0;
      const returnQty = parseFloat(document.getElementById('logReturn').value) || 0;

      if (!date || !prodName) {
        alert('Pumili muna ng petsa at produkto.');
        return;
      }

      if (!inventoryDailyLogs[date]) inventoryDailyLogs[date] = {};
      
      inventoryDailyLogs[date][prodName] = {
        qtyIn,
        returnQty
      };

      saveData();
      renderInventoryTables();
      document.getElementById('logQtyIn').value = '0';
      document.getElementById('logReturn').value = '0';
      alert('Na-save ang araw-araw na galaw para sa produktong ito!');
    }

    function getPreviousDateString(dateStr) {
      let d = new Date(dateStr);
      d.setDate(d.getDate() - 1);
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
    }

    function renderInventoryTables() {
      const selectedDate = document.getElementById('inventoryViewDate').value;
      const palmBody = document.getElementById('palmCocoInventoryBody');
      const otherBody = document.getElementById('otherInventoryBody');
      const productSelect = document.getElementById('logProductSelect');
      
      palmBody.innerHTML = '';
      otherBody.innerHTML = '';
      productSelect.innerHTML = `<option value="">-- Piliin ang Produkto mula sa Masterlist --</option>`;

      inventoryMasterList.forEach(p => {
        productSelect.innerHTML += `<option value="${p.name}">${p.name} (${p.description})</option>`;
      });

      if (inventoryMasterList.length === 0) {
        palmBody.innerHTML = `<tr><td colspan="8" class="text-center text-muted">Wala pang produkto sa Masterlist. Mag-add sa itaas.</td></tr>`;
        otherBody.innerHTML = `<tr><td colspan="8" class="text-center text-muted">Wala pang produkto sa Masterlist.</td></tr>`;
        return;
      }

      let soldMap = {};
      transactions.forEach(t => {
        if (t.itemsList && t.date === selectedDate) {
          t.itemsList.forEach(item => {
            soldMap[item.name.toLowerCase()] = (soldMap[item.name.toLowerCase()] || 0) + item.qty;
          });
        }
      });

      let dayLogs = (inventoryDailyLogs[selectedDate]) || {};
      let palmCount = 0;
      let otherCount = 0;

      inventoryMasterList.forEach(prod => {
        let log = dayLogs[prod.name] || { qtyIn: 0, returnQty: 0 };
        let soldQty = soldMap[prod.name.toLowerCase()] || 0;

        let beginning = 0;
        let prevDate = getPreviousDateString(selectedDate);
        if (window.cachedEnding && window.cachedEnding[prevDate] && window.cachedEnding[prevDate][prod.name]) {
          beginning = window.cachedEnding[prevDate][prod.name];
        }

        let ending = beginning + log.qtyIn + log.returnQty - soldQty;

        if (!window.cachedEnding) window.cachedEnding = {};
        if (!window.cachedEnding[selectedDate]) window.cachedEnding[selectedDate] = {};
        window.cachedEnding[selectedDate][prod.name] = ending;

        const rowHtml = `
          <tr>
            <td class="fw-bold">${prod.name}</td>
            <td>${prod.description}</td>
            <td>${prod.supplier}</td>
            <td>${beginning}</td>
            <td class="text-success">+${log.qtyIn}</td>
            <td class="text-warning">+${log.returnQty}</td>
            <td class="text-danger">-${soldQty}</td>
            <td class="fw-bold text-primary">${ending}</td>
          </tr>
        `;

        let isPalmCoco = prod.name.toLowerCase().includes('palm') || prod.name.toLowerCase().includes('coco') || prod.description.toLowerCase().includes('palm') || prod.description.toLowerCase().includes('coco');

        if (isPalmCoco) {
          palmCount++;
          palmBody.innerHTML += rowHtml;
        } else {
          otherCount++;
          otherBody.innerHTML += rowHtml;
        }
      });

      if (palmCount === 0) palmBody.innerHTML = `<tr><td colspan="8" class="text-center text-muted">Walang Palm & Coco item sa Masterlist.</td></tr>`;
      if (otherCount === 0) otherBody.innerHTML = `<tr><td colspan="8" class="text-center text-muted">Walang Iba pang item sa Masterlist.</td></tr>`;
    }

    // EXPENSES & SALARY LEDGER
    function addExpenseItem() {
      const date = document.getElementById('expenseDate').value;
      const type = document.getElementById('expenseType').value;
      const title = document.getElementById('expenseTitle').value.trim();
      const amount = parseFloat(document.getElementById('expenseAmount').value) || 0;

      if (!title || amount <= 0) {
        alert('Mangyaring ilagay ang wastong pamagat at halaga.');
        return;
      }

      if (!monthlyExpensesData[date]) monthlyExpensesData[date] = [];
      monthlyExpensesData[date].push({ id: Date.now(), type, title, amount });

      saveData();
      renderStandaloneExpensesLedger();
      document.getElementById('expenseTitle').value = '';
      document.getElementById('expenseAmount').value = '';
      alert('Matagumpay na naidagdag!');
    }

    function renderStandaloneExpensesLedger() {
      const tbody = document.getElementById('expensesTableBody');
      tbody.innerHTML = '';
      let count = 0;
      let totalSalary = 0;
      let totalExpenses = 0;

      Object.keys(monthlyExpensesData).sort().forEach(date => {
        monthlyExpensesData[date].forEach(ex => {
          count++;
          let isSalary = (ex.type === 'Salary' || (!ex.type && ex.title.toLowerCase().includes('sweldo')));
          let salVal = isSalary ? ex.amount : 0;
          let expVal = !isSalary ? ex.amount : 0;

          totalSalary += salVal;
          totalExpenses += expVal;

          tbody.innerHTML += `
            <tr>
              <td>${date}</td>
              <td class="fw-bold">${ex.title}</td>
              <td class="text-primary">${salVal > 0 ? '₱' + salVal.toFixed(2) : '-'}</td>
              <td class="text-danger">${expVal > 0 ? '₱' + expVal.toFixed(2) : '-'}</td>
              <td class="no-print text-center">
                <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteExpense('${date}', ${ex.id})"><i class="fa-solid fa-trash-can"></i></button>
              </td>
            </tr>
          `;
        });
      });

      if (count === 0) {
        tbody.innerHTML = `<tr><td colspan="5" class="text-center text-muted">Walang record na naitala.</td></tr>`;
      }
      document.getElementById('totalSalarySum').innerText = `₱${totalSalary.toFixed(2)}`;
      document.getElementById('totalExpensesSum').innerText = `₱${totalExpenses.toFixed(2)}`;
    }

    function deleteExpense(date, id) {
      if (confirm('Burahin ang record na ito?')) {
        monthlyExpensesData[date] = monthlyExpensesData[date].filter(ex => ex.id !== id);
        if (monthlyExpensesData[date].length === 0) delete monthlyExpensesData[date];
        saveData();
        renderStandaloneExpensesLedger();
      }
    }

    // D/ECO BOSS LEDGER
    function addBossItem() {
      const date = document.getElementById('bossDate').value;
      const title = document.getElementById('bossTitle').value.trim();
      const amount = parseFloat(document.getElementById('bossAmount').value) || 0;

      if (!title || amount <= 0) {
        alert('Ilagay ang detalye at halaga para kay Boss.');
        return;
      }

      if (!bossLedgerData[date]) bossLedgerData[date] = [];
      bossLedgerData[date].push({ id: Date.now(), title, amount });

      saveData();
      renderStandaloneBossLedger();
      document.getElementById('bossTitle').value = '';
      document.getElementById('bossAmount').value = '';
      alert('Matagumpay na nai-record kay Boss!');
    }

    function renderStandaloneBossLedger() {
      const tbody = document.getElementById('bossTableBody');
      tbody.innerHTML = '';
      let count = 0;

      Object.keys(bossLedgerData).sort().forEach(date => {
        bossLedgerData[date].forEach(b => {
          count++;
          tbody.innerHTML += `
            <tr>
              <td>${date}</td>
              <td class="fw-bold">${b.title}</td>
              <td class="text-success">₱${b.amount.toFixed(2)}</td>
              <td class="no-print text-center">
                <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteBossItem('${date}', ${b.id})"><i class="fa-solid fa-trash-can"></i></button>
              </td>
            </tr>
          `;
        });
      });

      if (count === 0) {
        tbody.innerHTML = `<tr><td colspan="4" class="text-center text-muted">Walang record para kay Boss.</td></tr>`;
      }
    }

    function deleteBossItem(date, id) {
      if (confirm('Burahin ang record na ito?')) {
        bossLedgerData[date] = bossLedgerData[date].filter(b => b.id !== id);
        if (bossLedgerData[date].length === 0) delete bossLedgerData[date];
        saveData();
        renderStandaloneBossLedger();
      }
    }

    // DAILY & MONTHLY AUDIT REVIEW PER DAY & MONTH
    function generateDailyMonthlyAudit() {
      const selectedDate = document.getElementById('auditDateInput').value;
      const selectedMonthPrefix = selectedDate ? selectedDate.substring(0, 7) : ''; // YYYY-MM

      // --- 1. ARAW NA ITO (DAILY SUMMARY) ---
      const tbody = document.getElementById('auditDailyDetailBody');
      tbody.innerHTML = '';
      let daySales = 0, dayProfit = 0, dayTotalExpAndSal = 0, dayBossSum = 0, count = 0;

      transactions.forEach(t => {
        if (t.date === selectedDate) {
          const cost = t.totalCost || 0;
          const profit = t.total - cost;
          daySales += t.total;
          dayProfit += profit;
          count++;

          tbody.innerHTML += `
            <tr>
              <td><span class="badge bg-primary">Benta (Sale)</span></td>
              <td class="fw-bold">${t.customer} (${t.product})</td>
              <td>${t.location}</td>
              <td>-</td>
              <td>-</td>
              <td>-</td>
              <td class="text-success">₱${profit.toFixed(2)}</td>
            </tr>
          `;
        }
      });

      if (monthlyExpensesData[selectedDate]) {
        monthlyExpensesData[selectedDate].forEach(ex => {
          let isSalary = (ex.type === 'Salary' || (!ex.type && ex.title.toLowerCase().includes('sweldo')));
          let salVal = isSalary ? ex.amount : 0;
          let expVal = !isSalary ? ex.amount : 0;
          dayTotalExpAndSal += (salVal + expVal);
          count++;

          tbody.innerHTML += `
            <tr>
              <td><span class="badge ${isSalary ? 'bg-info text-dark' : 'bg-danger'}">${ex.type || (isSalary ? 'Salary' : 'Expense')}</span></td>
              <td class="fw-bold">${ex.title}</td>
              <td>Salary / Expense</td>
              <td class="text-primary">${salVal > 0 ? '-₱' + salVal.toFixed(2) : '-'}</td>
              <td class="text-danger">${expVal > 0 ? '-₱' + expVal.toFixed(2) : '-'}</td>
              <td>-</td>
              <td>-</td>
            </tr>
          `;
        });
      }

      if (bossLedgerData[selectedDate]) {
        bossLedgerData[selectedDate].forEach(b => {
          dayBossSum += (b.amount || 0);
          count++;

          tbody.innerHTML += `
            <tr>
              <td><span class="badge bg-success">D/Eco Boss</span></td>
              <td class="fw-bold">${b.title}</td>
              <td>D/Eco Boss Ledger</td>
              <td>-</td>
              <td>-</td>
              <td class="text-success">+₱${b.amount.toFixed(2)}</td>
              <td>-</td>
            </tr>
          `;
        });
      }

      if (count === 0) {
        tbody.innerHTML = `<tr><td colspan="7" class="text-center text-muted py-3">Walang record sa petsang ito.</td></tr>`;
      }

      const netIncome = (dayProfit - dayTotalExpAndSal) + dayBossSum;

      document.getElementById('auditDaySales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('auditDayProfit').innerText = `₱${dayProfit.toFixed(2)}`;
      document.getElementById('auditDayExpenses').innerText = `₱${dayTotalExpAndSal.toFixed(2)}`;
      document.getElementById('auditDayNetIncome').innerText = `₱${netIncome.toFixed(2)}`;


      // --- 2. KABUUANG BUWAN (MONTHLY SUMMARY) ---
      const monthTbody = document.getElementById('auditMonthlyDetailBody');
      monthTbody.innerHTML = '';
      let monthSales = 0, monthProfit = 0, monthSalSum = 0, monthExpSum = 0, monthBossSum = 0, monthItemCount = 0;

      // Kolektahin at ayusin ang lahat ng petsa sa napiling buwan
      let allDatesInMonth = new Set();
      transactions.forEach(t => { if (t.date && t.date.startsWith(selectedMonthPrefix)) allDatesInMonth.add(t.date); });
      Object.keys(monthlyExpensesData).forEach(d => { if (d.startsWith(selectedMonthPrefix)) allDatesInMonth.add(d); });
      Object.keys(bossLedgerData).forEach(d => { if (d.startsWith(selectedMonthPrefix)) allDatesInMonth.add(d); });

      let sortedDates = Array.from(allDatesInMonth).sort();

      sortedDates.forEach(dt => {
        // 1. Transactions sa petsang ito
        transactions.forEach(t => {
          if (t.date === dt) {
            const cost = t.totalCost || 0;
            const profit = t.total - cost;
            monthSales += t.total;
            monthProfit += profit;
            monthItemCount++;

            monthTbody.innerHTML += `
              <tr>
                <td>${dt}</td>
                <td><span class="badge bg-primary">Benta</span></td>
                <td class="fw-bold">${t.customer} (${t.product})</td>
                <td>${t.location}</td>
                <td>-</td>
                <td>-</td>
                <td>-</td>
                <td class="text-success">₱${profit.toFixed(2)}</td>
              </tr>
            `;
          }
        });

        // 2. Expenses / Salary sa petsang ito
        if (monthlyExpensesData[dt]) {
          monthlyExpensesData[dt].forEach(ex => {
            let isSalary = (ex.type === 'Salary' || (!ex.type && ex.title.toLowerCase().includes('sweldo')));
            let salVal = isSalary ? ex.amount : 0;
            let expVal = !isSalary ? ex.amount : 0;
            monthSalSum += salVal;
            monthExpSum += expVal;
            monthItemCount++;

            monthTbody.innerHTML += `
              <tr>
                <td>${dt}</td>
                <td><span class="badge ${isSalary ? 'bg-info text-dark' : 'bg-danger'}">${ex.type || (isSalary ? 'Salary' : 'Expense')}</span></td>
                <td class="fw-bold">${ex.title}</td>
                <td>Salary / Expense</td>
                <td class="text-primary">${salVal > 0 ? '-₱' + salVal.toFixed(2) : '-'}</td>
                <td class="text-danger">${expVal > 0 ? '-₱' + expVal.toFixed(2) : '-'}</td>
                <td>-</td>
                <td>-</td>
              </tr>
            `;
          });
        }

        // 3. D/Eco Boss sa petsang ito
        if (bossLedgerData[dt]) {
          bossLedgerData[dt].forEach(b => {
            monthBossSum += (b.amount || 0);
            monthItemCount++;

            monthTbody.innerHTML += `
              <tr>
                <td>${dt}</td>
                <td><span class="badge bg-success">D/Eco Boss</span></td>
                <td class="fw-bold">${b.title}</td>
                <td>D/Eco Boss Ledger</td>
                <td>-</td>
                <td>-</td>
                <td class="text-success">+₱${b.amount.toFixed(2)}</td>
                <td>-</td>
              </tr>
            `;
          });
        }
      });

      if (monthItemCount === 0) {
        monthTbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Walang record sa buwang ito (${selectedMonthPrefix}).</td></tr>`;
      }

      const totalMonthExpAndSal = monthSalSum + monthExpSum;
      const monthNetIncome = (monthProfit - totalMonthExpAndSal) + monthBossSum;

      document.getElementById('auditMonthSales').innerText = `₱${monthSales.toFixed(2)}`;
      document.getElementById('auditMonthProfit').innerText = `₱${monthProfit.toFixed(2)}`;
      document.getElementById('auditMonthExpenses').innerText = `₱${totalMonthExpAndSal.toFixed(2)}`;
      document.getElementById('auditMonthNetIncome').innerText = `₱${monthNetIncome.toFixed(2)}`;

      document.getElementById('auditMonthTotalSal').innerText = `₱${monthSalSum.toFixed(2)}`;
      document.getElementById('auditMonthTotalExp').innerText = `₱${monthExpSum.toFixed(2)}`;
      document.getElementById('auditMonthTotalBoss').innerText = `₱${monthBossSum.toFixed(2)}`;
      document.getElementById('auditMonthTotalProfit').innerText = `₱${monthProfit.toFixed(2)}`;
    }

    function searchCustomerOrder() {
      const query = document.getElementById('searchCustomerInput').value.trim().toLowerCase();
      const container = document.getElementById('searchResultContainer');
      const notFound = document.getElementById('noCustomerFound');
      
      if (!query) { 
        container.style.display = 'none'; 
        notFound.classList.add('d-none'); 
        return; 
      }

      const matched = transactions.filter(t => t.customer.toLowerCase().includes(query));
      
      if (matched.length > 0) {
        notFound.classList.add('d-none');
        container.style.display = 'block';

        const lastOrder = matched[matched.length - 1];
        document.getElementById('lastOrderCustomer').innerText = lastOrder.customer;
        document.getElementById('lastOrderDate').innerText = lastOrder.date;
        document.getElementById('lastOrderLocation').innerText = lastOrder.location;
        document.getElementById('lastOrderTotal').innerText = `₱${lastOrder.total.toFixed(2)}`;
        document.getElementById('lastOrderPaid').innerText = `₱${lastOrder.paid.toFixed(2)}`;
        document.getElementById('lastOrderBalance').innerText = `₱${lastOrder.balance.toFixed(2)}`;
        
        const badge = document.getElementById('lastOrderBadge');
        badge.innerText = lastOrder.status;
        badge.className = lastOrder.status === 'PAID' ? 'badge bg-success fs-6' : 'badge bg-warning text-dark fs-6';

        const itemsBody = document.getElementById('lastOrderItemsBody');
        itemsBody.innerHTML = '';
        if (lastOrder.itemsList && lastOrder.itemsList.length > 0) {
          lastOrder.itemsList.forEach(item => {
            itemsBody.innerHTML += `
              <tr>
                <td>${item.name} ${item.desc ? `(${item.desc})` : ''}</td>
                <td>${item.qty}</td>
                <td>₱${(item.price || 0).toFixed(2)}</td>
                <td>₱${(item.subtotal || (item.qty * item.price)).toFixed(2)}</td>
              </tr>
            `;
          });
        } else {
          itemsBody.innerHTML = `<tr><td colspan="4" class="text-center text-muted">(${lastOrder.product})</td></tr>`;
        }

        const historyBody = document.getElementById('customerHistoryBody');
        historyBody.innerHTML = '';
        matched.forEach(t => {
          historyBody.innerHTML += `
            <tr>
              <td>${t.date}</td>
              <td><span class="badge bg-secondary">${t.location}</span></td>
              <td>${t.product}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger">₱${t.balance.toFixed(2)}</td>
              <td><span class="badge ${t.status === 'PAID' ? 'bg-success' : 'bg-warning text-dark'}">${t.status}</span></td>
            </tr>
          `;
        });

      } else {
        container.style.display = 'none';
        notFound.classList.remove('d-none');
      }
    }

    function deleteTransaction(id) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang transaksyong ito?')) {
        transactions = transactions.filter(t => t.id !== id);
        saveData();
        generateDailyReport();
        renderCreditTable();
        renderInventoryTables();
      }
    }
  </script>
</body>
</html>
