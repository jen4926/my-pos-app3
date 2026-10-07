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
    .inventory-input { width: 95px; text-align: center; }
    .stat-card { border-left: 4px solid #1976d2; }
   
    /* Login Backdrop overlay */
    #loginOverlay {
      position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
      background: rgba(13, 71, 161, 0.85); z-index: 9999;
      display: flex; justify-content: center; align-items: center;
    }

    /* PRINT STYLES */
    @media print {
      body {
        background-color: #fff !important;
        color: #000 !important;
        font-size: 12pt;
      }
      .navbar, #loginOverlay, .btn, .nav, .modal, .no-print {
        display: none !important;
      }
      .card {
        border: none !important;
        box-shadow: none !important;
        padding: 0 !important;
      }
      .container {
        max-width: 100% !important;
        padding: 0 !important;
        margin: 0 !important;
      }
      .tab-pane {
        display: block !important;
        opacity: 1 !important;
      }
    }
  </style>
</head>
<body>

  <!-- ================= 0. LOGIN OVERLAY ================= -->
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
          <input type="text" id="loginUsername" class="form-control" placeholder="e.g. admin" required>
        </div>
        <div class="mb-3">
          <label class="form-label fw-semibold">Password:</label>
          <input type="password" id="loginPassword" class="form-control" placeholder="••••••••" required>
        </div>
        <div id="loginError" class="alert alert-danger p-2 small d-none">
          Mali ang username o password!
        </div>
        <button type="submit" class="btn btn-primary w-100 fw-bold py-2"><i class="fa-solid fa-right-to-bracket me-2"></i>Log In</button>
      </form>
    </div>
  </div>

  <!-- Navbar -->
  <nav class="navbar navbar-dark expand-lg mb-4">
    <div class="container-fluid">
      <a class="navbar-brand fw-bold fs-4" href="#">
        <i class="fa-solid fa-store me-2"></i>RMVillasis Enterprises
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
          <button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button" onclick="renderCreditTable(); renderStandalonePayments();">
            <i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="search-tab" data-bs-toggle="pill" data-bs-target="#search-content" type="button">
            <i class="fa-solid fa-magnifying-glass me-1"></i> Order Lookup & Last Price
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTables(); renderStockInHistory(); renderCustomerSalesLog(); renderDailyInventorySheet();">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="expenses-tab" data-bs-toggle="pill" data-bs-target="#expenses-content" type="button" onclick="renderStandaloneExpensesLedger()">
            <i class="fa-solid fa-receipt me-1"></i> Salary & Expenses
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="boss-tab" data-bs-toggle="pill" data-bs-target="#boss-content" type="button" onclick="renderStandaloneBossLedger()">
            <i class="fa-solid fa-user-tie me-1"></i> D/Eco Boss
          </button>
        </li>
        <li class="nav-item admin-only">
          <button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()">
            <i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit
          </button>
        </li>
      </ul>

      <!-- User Profile, Save, Refresh Button & Account Controls -->
      <div class="d-flex align-items-center gap-2">
        <button class="btn btn-success btn-sm fw-semibold" onclick="manualSaveData()" title="Save Data to Local Storage">
          <i class="fa-solid fa-floppy-disk me-1"></i> Save Data
        </button>
        <button class="btn btn-warning btn-sm fw-semibold text-dark" onclick="exportDataBackup()" title="Download Backup File">
          <i class="fa-solid fa-download me-1"></i> Download Backup
        </button>
        <label class="btn btn-info btn-sm fw-semibold text-white mb-0" title="Restore Data from Backup File">
          <i class="fa-solid fa-upload me-1"></i> Restore <input type="file" id="importFile" accept=".json" onchange="importDataBackup(event)" style="display: none;">
        </label>
        <button class="btn btn-outline-light btn-sm fw-semibold" onclick="location.reload()" title="Refresh Page">
          <i class="fa-solid fa-rotate me-1"></i> Refresh
        </button>
        <div class="dropdown text-end text-white">
          <a href="#" class="d-block link-light text-decoration-none dropdown-toggle fw-bold" id="userDropdown" data-bs-toggle="dropdown">
            <i class="fa-solid fa-circle-user fa-lg me-1"></i> <span id="currentUserName">User</span>
          </a>
          <ul class="dropdown-menu dropdown-menu-end text-small shadow">
            <li><a class="dropdown-item" href="#" onclick="openChangeProfileModal()"><i class="fa-solid fa-key me-2"></i>Change Name / Password</a></li>
            <li class="admin-only"><a class="dropdown-item" href="#" onclick="openUserManagementModal()"><i class="fa-solid fa-users-gear me-2"></i>Manage Users & Admins</a></li>
            <li><hr class="dropdown-divider"></li>
            <li><a class="dropdown-item text-danger fw-bold" href="#" onclick="logout()"><i class="fa-solid fa-right-from-bracket me-2">Tag Out</i></a></li>
          </ul>
        </div>
      </div>
    </div>
  </nav>

  <div class="container pb-5">
    <div class="tab-content" id="mainTabsContent">

      <!-- ================= 1. POS ENTRY TAB ================= -->
      <div class="tab-pane fade show active" id="pos-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-cash-register me-2"></i>Record New Transaction (Puwede ring ilagay ang petsa ng nakaraang araw)</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> I-print ang Page / Resibo
            </button>
          </div>
         
          <!-- Inventory Only Mode Toggle -->
          <div class="alert alert-info py-2 mb-3 d-flex justify-content-between align-items-center">
            <div>
              <i class="fa-solid fa-info-circle me-1"></i> <strong>Mode:</strong> Pwede kang magpasok ng item para sa Inventory Lang (Walang halaga/presyo) para sa nakaraang petsa.
            </div>
            <div class="form-check form-switch mb-0">
              <input class="form-check-input" type="checkbox" id="inventoryOnlyMode" onchange="toggleInventoryOnlyMode()">
              <label class="form-check-label fw-semibold" for="inventoryOnlyMode">Inventory Only Mode (No Price/Cost)</label>
            </div>
          </div>

          <form id="posForm">
            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Date of Sale / Log:</label>
                <input type="date" id="saleDate" class="form-control" required>
                <small class="text-muted">Palitan ang petsa kung ito ay para sa nakaraang araw.</small>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer / Reference Name:</label>
                <input type="text" id="customerName" class="form-control" placeholder="e.g., Juan Dela Cruz o Stock In" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Transaction Location / Uri:</label>
                <select id="transactionLocation" class="form-select" required>
                  <option value="Hiway">Hiway</option>
                  <option value="Byahe">Byahe</option>
                  <option value="Inventory Only">Inventory Only / Stock Update</option>
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
                    <th style="width: 130px;" class="price-col">Cost / Unit (₱)</th>
                    <th style="width: 130px;" class="price-col">Price / Unit (₱)</th>
                    <th style="width: 130px;" class="price-col">Subtotal (₱)</th>
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

            <!-- CONTAINER / DEPOSIT DETAILS -->
            <div class="card p-3 bg-light border mb-3" id="containerSectionBox">
              <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-box-open me-2"></i>Container / Lalagyan Details</h6>
              <div class="row g-3">
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Container Status:</label>
                  <select id="containerStatus" class="form-select" onchange="toggleContainerFields()">
                    <option value="NONE">Walang Container / Soli Agad</option>
                    <option value="HIRAM">Hiram / Bagon (Walang Deposit)</option>
                    <option value="DEPOSIT">May Deposito (With Deposit Fee)</option>
                  </select>
                </div>
                <div class="col-md-4 container-qty-group" style="display: none;">
                  <label class="form-label fw-semibold">Ilang Container / Lalagyan:</label>
                  <input type="number" step="any" min="0.5" id="containerQty" class="form-control" value="1" placeholder="Hal. 2">
                </div>
                <div class="col-md-4 container-deposit-group" style="display: none;">
                  <label class="form-label fw-semibold">Halaga ng Deposito bawat Isa (₱):</label>
                  <input type="number" step="0.01" min="0" id="containerDepositRate" class="form-control" placeholder="0.00" oninput="calculateTotal()">
                </div>
              </div>
            </div>

            <div class="row g-3 mt-2" id="financialSectionBox">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Total Amount (₱) <small class="text-muted">(Inc. Deposit)</small>:</label>
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

            <!-- MULTI-PAYMENT BREAKDOWN SECTION -->
            <div id="multiPaymentFields" class="multi-payment-fields">
              <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-wallet me-2"></i>Hati ng Bayad (Multi-Payment Breakdown)</h6>
              <p class="text-muted small mb-2">Ilagay kung magkano ang napunta sa bawat uri ng bayad. Ang Cash portion lamang ang awtomatikong isasama sa Drawer Cash Target.</p>
              <div class="row g-2">
                <div class="col-md-3">
                  <label class="form-label small fw-semibold">Cash Amount (₱):</label>
                  <input type="number" step="0.01" id="multiCashAmt" class="form-control form-control-sm" placeholder="0.00" value="0.00">
                </div>
                <div class="col-md-3">
                  <label class="form-label small fw-semibold">GCash Amount (₱):</label>
                  <input type="number" step="0.01" id="multiGcashAmt" class="form-control form-control-sm" placeholder="0.00" value="0.00">
                </div>
                <div class="col-md-3">
                  <label class="form-label small fw-semibold">Bank Transfer / BT (₱):</label>
                  <input type="number" step="0.01" id="multiBTAmt" class="form-control form-control-sm" placeholder="0.00" value="0.00">
                </div>
                <div class="col-md-3">
                  <label class="form-label small fw-semibold">Byahe Cash / Iba pa (₱):</label>
                  <input type="number" step="0.01" id="multiOtherAmt" class="form-control form-control-sm" placeholder="0.00" value="0.00">
                </div>
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
              <button type="submit" class="btn btn-success btn-lg px-4"><i class="fa-solid fa-check me-2"></i>Save Transaction / Inventory</button>
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
              <button class="btn btn-outline-primary ms-2" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Report
              </button>
            </div>
          </div>

          <div class="row g-3 mb-4">
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">HIWAY SALES / PROFIT</span>
                <div class="mt-1">
                  <span class="text-primary fw-bold" id="dailyHiwaySales">₱0.00</span> <small class="text-muted">(Sales)</small><br>
                  <span class="text-success small fw-bold" id="dailyHiwayProfit">₱0.00</span> <small class="text-muted">(Net)</small>
                </div>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;">
                <span class="text-muted small fw-bold">BYAHE SALES / PROFIT</span>
                <div class="mt-1">
                  <span class="text-primary fw-bold" id="dailyByaheSales">₱0.00</span> <small class="text-muted">(Sales)</small><br>
                  <span class="text-success small fw-bold" id="dailyByaheProfit">₱0.00</span> <small class="text-muted">(Net)</small>
                </div>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">DAILY REAL NET PROFIT (Less Exp/Sahod)</span>
                <div class="mt-1">
                  <span class="text-success fw-bold fs-5" id="dailyTotalNetProfit">₱0.00</span> <small class="text-muted">(Net Kita)</small><br>
                  <span class="text-secondary small" id="dailyExpensesDeducted">Exp: ₱0.00</span>
                </div>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;">
                <span class="text-muted small fw-bold">TOTAL SALES & TX</span>
                <div class="mt-1">
                  <span class="text-primary fw-bold" id="dailyTotalSales">₱0.00</span> <small class="text-muted">(Total)</small><br>
                  <span class="text-warning fw-bold" id="dailyTxCount">0</span> <small class="text-muted">(Count)</small>
                </div>
              </div>
            </div>
          </div>

          <!-- MONEY BREAKDOWN & AUDIT SECTION -->
          <div class="card p-3 bg-light border mb-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-money-bill-wave me-2"></i>Daily Cash Money Breakdown (Denominations)</h6>
              <button type="button" class="btn btn-sm btn-outline-secondary" onclick="clearMoneyBreakdown()">
                <i class="fa-solid fa-rotate-right me-1"></i> I-refresh / Clear Breakdown
              </button>
            </div>
            <div class="row g-3">
              <div class="col-md-7">
                <div class="table-responsive">
                  <table class="table table-sm table-bordered bg-white align-middle m-0">
                    <thead class="table-dark">
                      <tr>
                        <th>Denomination</th>
                        <th style="width: 130px;">Count / Pcs</th>
                        <th style="width: 150px;">Subtotal (₱)</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td class="fw-semibold text-primary">₱1,000</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="1000" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱500</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="500" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱200</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="200" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱100</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="100" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱50</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="50" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱20</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="20" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">Coins / Barya</td>
                        <td><span class="text-muted small">Kabuuang Barya</span></td>
                        <td><input type="number" step="0.01" min="0" class="form-control form-control-sm denom-coins" placeholder="0.00" oninput="calculateMoneyBreakdown()" onkeydown="handleEnterNext(event, this)"></td>
                      </tr>
                    </tbody>
                    <tfoot class="table-secondary fw-bold">
                      <tr>
                        <td class="text-end">TOTAL KABUUANG PERA SA DRAWER:</td>
                        <td id="breakdownTotalPcs" class="text-center">0 pcs</td>
                        <td id="breakdownTotalAmount" class="text-success">₱0.00</td>
                      </tr>
                    </tfoot>
                  </table>
                </div>
              </div>

              <!-- CASH VERIFICATION / DISCREPANCY COMPARISON -->
              <div class="col-md-5 d-flex flex-column justify-content-between">
                <div class="card p-3 bg-white h-100 border">
                  <h6 class="fw-bold text-dark border-bottom pb-2 mb-3"><i class="fa-solid fa-scale-balanced me-2"></i>Cash Audit & Deductions</h6>
                 
                  <div class="mb-3 bg-warning-subtle p-2 rounded border border-warning">
                    <label class="form-label fw-bold text-dark small mb-1"><i class="fa-solid fa-wallet me-1"></i> Pondo / Change Fund (Idaragdag):</label>
                    <input type="number" step="0.01" min="0" class="form-control form-control-sm fw-bold denom-fund bg-white" id="cashFundInput" placeholder="0.00" oninput="calculateMoneyBreakdown()">
                  </div>

                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small fw-semibold">Total Collections (Sales):</span>
                    <span class="fw-bold text-secondary" id="totalCollectionAll">₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small fw-semibold text-success">(+) Payment sa Utang (Collected):</span>
                    <span class="fw-bold text-success" id="breakdownDebtPayment">+₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: Byahe Cash</span>
                    <span class="text-danger small" id="lessByaheCash">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: GCash</span>
                    <span class="text-danger small" id="lessGCash">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: Bank Transfer (BT)</span>
                    <span class="text-danger small" id="lessBT">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: Cheque</span>
                    <span class="text-danger small" id="lessCheque">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small text-danger">Less: Salary & Expenses (Cash Out):</span>
                    <span class="text-danger small" id="lessExpenses">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small text-danger">Less: Remaining Balance (Unpaid Credit):</span>
                    <span class="text-danger small" id="lessRemainingBalance">-₱0.00</span>
                  </div>

                  <hr class="my-1">

                  <div class="d-flex justify-content-between align-items-center my-2 bg-light p-2 rounded">
                    <span class="fw-bold text-dark">Target Cash in Drawer (Sales + Payment):</span>
                    <span class="fs-6 fw-bold text-success" id="breakdownTargetSales">₱0.00</span>
                  </div>

                  <div class="d-flex justify-content-between align-items-center mb-2 bg-warning-subtle p-2 rounded border border-warning-subtle">
                    <span class="fw-bold text-dark small"><i class="fa-solid fa-wallet me-1"></i> Target Cash + Pondo:</span>
                    <span class="fs-5 fw-bold text-primary" id="breakdownTargetWithFund">₱0.00</span>
                  </div>

                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="text-muted fw-semibold">Total Cash Counted (Minus Pondo):</span>
                    <span class="fs-5 fw-bold text-dark" id="totalCountedCash">₱0.00</span>
                  </div>

                  <div class="d-flex justify-content-between align-items-center mb-3">
                    <span class="fw-bold text-dark">Discrepancy / Over-Short:</span>
                    <span class="fs-5 fw-bold" id="cashDiscrepancy">₱0.00</span>
                  </div>

                  <div id="cashStatusAlert" class="alert alert-secondary text-center p-2 fw-bold mb-0">
                    <i class="fa-solid fa-calculator me-1"></i> Magpasok ng breakdown para ma-audit.
                  </div>
                </div>
              </div>
            </div>
          </div>

          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>List of Encoded Transactions (Para sa Napiling Petsa)</h6>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>#</th>
                  <th>Customer Name</th>
                  <th>Location</th>
                  <th>Products Bought</th>
                  <th>Container Status</th>
                  <th>Total Cost (₱)</th>
                  <th>Total Amount</th>
                  <th>Paid Amount</th>
                  <th>Balance</th>
                  <th>Net Profit (₱)</th>
                  <th>Payment Method</th>
                  <th class="text-center no-print" style="width: 100px;">Actions</th>
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
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-users-viewfinder me-2"></i>Customer Credit & Utang Ledger</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-outline-secondary" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Utang List
              </button>
              <button class="btn btn-success fw-bold" data-bs-toggle="modal" data-bs-target="#standalonePaymentModal">
                <i class="fa-solid fa-plus me-1"></i> Add Manual Payment / Collection
              </button>
            </div>
          </div>

          <!-- STANDALONE MANUAL PAYMENT / COLLECTION LEDGER TABLE -->
          <div class="card p-3 bg-light border mb-4">
            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-receipt me-2"></i>Standalone Manual Payments & Collections History (Araw-arawang Bayad sa Lumang Utang)</h6>
            <div class="input-group mb-3">
              <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
              <input type="text" id="searchStandalonePaymentInput" class="form-control" placeholder="I-search ang pangalan ng customer o petsa..." oninput="renderStandalonePayments()">
            </div>
            <div class="table-responsive">
              <table class="table table-bordered align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Date (Petsa)</th>
                    <th>Customer Name</th>
                    <th>Payment Method</th>
                    <th class="text-end">Amount Paid (₱)</th>
                    <th>Notes / Remarks</th>
                    <th class="col-action no-print text-center"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="standalonePaymentTableBody">
                  <!-- Dynamic Standalone Payment Rows -->
                </tbody>
                <tfoot class="table-secondary fw-bold" id="standalonePaymentTableFooter">
                  <!-- Total Subtotal -->
                </tfoot>
              </table>
            </div>
          </div>

          <div class="table-responsive mb-5">
            <table class="table table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>Customer Name</th>
                  <th>Location</th>
                  <th>Product(s)</th>
                  <th>Total Cost (₱)</th>
                  <th>Amount Paid (₱)</th>
                  <th>Balance (₱)</th>
                  <th>Due Date</th>
                  <th>Status</th>
                  <th class="no-print">Action</th>
                </tr>
              </thead>
              <tbody id="creditTableBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>

          <div class="border-top pt-4">
            <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>Listahan ng mga Nakapagbayad na (Paid Accounts History)</h5>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Customer Name</th>
                    <th>Product(s)</th>
                    <th>Total Amount (₱)</th>
                    <th>Total Paid (₱)</th>
                    <th>Status</th>
                    <th class="text-center no-print" style="width: 120px;">Actions</th>
                  </tr>
                </thead>
                <tbody id="paidHistoryTableBody">
                  <!-- Dynamic Content -->
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 3.5 CUSTOMER ORDER LOOKUP & LAST PRICE TAB ================= -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-magnifying-glass me-2"></i>Track Customer Last Order & Last Price Review</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> Print Customer Record
            </button>
          </div>
         
          <div class="row g-2 mb-4">
            <div class="col-md-9">
              <input type="text" id="searchCustomerInput" class="form-control form-control-lg" placeholder="I-type ang pangalan ng Customer (e.g. Juan Dela Cruz)">
            </div>
            <div class="col-md-3">
              <button class="btn btn-primary btn-lg w-100 fw-bold" onclick="searchCustomerOrder()">
                <i class="fa-solid fa-search me-2"></i>Search Order & Price
              </button>
            </div>
          </div>

          <div id="searchResultContainer" style="display: none;">
            <div class="card bg-light border-primary mb-4">
              <div class="card-header bg-primary text-white fw-bold d-flex justify-content-between align-items-center">
                <span><i class="fa-solid fa-receipt me-2"></i>Huling Order at Huling Presyo (Last Order & Last Price Review)</span>
                <span id="lastOrderBadge" class="badge bg-warning text-dark fs-6">Status</span>
              </div>
              <div class="card-body">
                <div class="row g-3">
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">CUSTOMER NAME:</p>
                    <h5 class="fw-bold text-dark" id="lastOrderCustomer">-</h5>
                  </div>
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">DATE OF PURCHASE:</p>
                    <h5 class="fw-bold text-dark" id="lastOrderDate">-</h5>
                  </div>
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">CONTAINER STATUS:</p>
                    <h5 class="fw-bold text-dark" id="lastOrderContainer">-</h5>
                  </div>
                  <div class="col-md-12">
                    <p class="mb-1 text-muted small fw-bold">DETALYADONG LISTAHAN NG HULING BINILI (Items, Qty & Last Price):</p>
                    <div class="table-responsive">
                      <table class="table table-sm table-bordered bg-white align-middle" id="lastOrderItemsTable">
                        <thead class="table-light">
                          <tr>
                            <th>Product Name</th>
                            <th>Description</th>
                            <th class="text-center">Qty</th>
                            <th class="text-end">Last Price / Unit (₱)</th>
                            <th class="text-end">Subtotal (₱)</th>
                          </tr>
                        </thead>
                        <tbody id="lastOrderItemsBody">
                          <!-- Dynamic Last Order Items -->
                        </tbody>
                      </table>
                    </div>
                  </div>
                  <hr class="my-2">
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">TOTAL AMOUNT:</p>
                    <h4 class="fw-bold text-primary" id="lastOrderTotal">₱0.00</h4>
                  </div>
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">PAID AMOUNT:</p>
                    <h4 class="fw-bold text-success" id="lastOrderPaid">₱0.00</h4>
                  </div>
                  <div class="col-md-4">
                    <p class="mb-1 text-muted small fw-bold">REMAINING BALANCE:</p>
                    <h4 class="fw-bold text-danger" id="lastOrderBalance">₱0.00</h4>
                  </div>
                </div>
              </div>
            </div>

            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-clock-rotate-left me-2"></i>Lahat ng Naging Transaksyon at Na-encode na Detalye ni Customer (Complete History)</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark">
                  <tr>
                    <th>Date</th>
                    <th>Location</th>
                    <th>Products & Price Details</th>
                    <th>Container</th>
                    <th>Total (₱)</th>
                    <th>Paid (₱)</th>
                    <th>Balance (₱)</th>
                    <th>Net Profit (₱)</th>
                    <th>Status</th>
                    <th class="text-center no-print" style="width: 150px;">Actions</th>
                  </tr>
                </thead>
                <tbody id="customerHistoryBody">
                  <!-- Dynamic History Rows -->
                </tbody>
              </table>
            </div>
          </div>

          <div id="noCustomerFound" class="alert alert-warning text-center p-3 d-none">
            <i class="fa-solid fa-triangle-exclamation me-2"></i> Walang nahanap na record para sa customer na ito.
          </div>
        </div>
      </div>

      <!-- ================= 4. INVENTORY TAB ================= -->
      <div class="tab-pane fade" id="inventory-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory Management & Product Cost</h4>
            <div class="d-flex gap-2">
              <button class="btn btn-outline-secondary" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Inventory
              </button>
              <button class="btn btn-info text-white fw-bold me-1" data-bs-toggle="modal" data-bs-target="#returnModal">
                <i class="fa-solid fa-rotate-left me-1"></i> Item Return / Isauli
              </button>
              <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#addProductModal">
                <i class="fa-solid fa-plus me-1"></i> Add Product
              </button>
            </div>
          </div>

          <!-- PER-DAY INVENTORY SHEET VIEW -->
          <div class="card p-3 bg-light border mb-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="fw-bold text-secondary m-0"><i class="fa-solid fa-calendar-day me-2"></i>Per-Day Inventory Sheet (Beginning, Out/Sold & Ending)</h5>
              <div class="d-flex align-items-center gap-2">
                <label class="fw-bold small">Piliin ang Araw (Date):</label>
                <input type="date" id="inventorySheetDate" class="form-control form-control-sm" onchange="renderDailyInventorySheet()">
              </div>
            </div>
           
            <!-- Per-Day Table 1: Palm & Coco -->
            <h6 class="fw-bold text-primary mb-2">Palm & Coco Inventory Sheet</h6>
            <div class="table-responsive mb-4">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark text-center">
                  <tr>
                    <th class="text-start">Product Name</th>
                    <th>Description</th>
                    <th>Beginning Stock</th>
                    <th>Stock In (+Add)</th>
                    <th>Return / Isauli</th>
                    <th>Total Out (Sold)</th>
                    <th class="table-success">Ending Stock (Lilipat Bukas)</th>
                  </tr>
                </thead>
                <tbody id="dailyPalmCocoSheetBody">
                  <!-- Dynamic Palm & Coco Per-Day Rows -->
                </tbody>
              </table>
            </div>

            <!-- Per-Day Table 2: Dedicated Products -->
            <h6 class="fw-bold text-primary mb-2">Dedicated Products Inventory Sheet</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark text-center">
                  <tr>
                    <th class="text-start">Product Name</th>
                    <th>Description</th>
                    <th>Beginning Stock</th>
                    <th>Stock In (+Add)</th>
                    <th>Return / Isauli</th>
                    <th>Total Out (Sold)</th>
                    <th class="table-success">Ending Stock (Lilipat Bukas)</th>
                  </tr>
                </thead>
                <tbody id="dailyDedicatedSheetBody">
                  <!-- Dynamic Dedicated Products Per-Day Rows -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- Master Inventory Table 1: Palm & Coco -->
          <h5 class="fw-bold text-primary mb-2">Palm & Coco Inventory Master List</h5>
          <div class="table-responsive mb-4">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark text-center">
                <tr>
                  <th class="text-start">Product Name</th>
                  <th>Description</th>
                  <th>Cost / Unit (₱)</th>
                  <th>Price / Unit (₱)</th>
                  <th>Beginning Stock</th>
                  <th>Stock In (+Add)</th>
                  <th>Sold</th>
                  <th>Ending Stock</th>
                  <th class="col-action no-print"><i class="fa-solid fa-trash"></i></th>
                </tr>
              </thead>
              <tbody id="palmCocoInventoryTableBody">
                <!-- Dynamic Palm & Coco Content -->
              </tbody>
            </table>
          </div>

          <!-- Master Inventory Table 2: Dedicated Products List -->
          <h5 class="fw-bold text-primary mb-2">Dedicated Products Inventory Master List</h5>
          <div class="table-responsive mb-4">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark text-center">
                <tr>
                  <th class="text-start">Product Name</th>
                  <th>Description</th>
                  <th>Cost / Unit (₱)</th>
                  <th>Price / Unit (₱)</th>
                  <th>Beginning Stock</th>
                  <th>Stock In (+Add)</th>
                  <th>Sold</th>
                  <th>Ending Stock</th>
                  <th class="col-action no-print"><i class="fa-solid fa-trash"></i></th>
                </tr>
              </thead>
              <tbody id="dedicatedInventoryTableBody">
                <!-- Dynamic Dedicated Products Content -->
              </tbody>
              <tfoot class="table-secondary fw-bold text-center" id="inventoryTableFooter">
                <!-- Total Row rendered dynamically -->
              </tfoot>
            </table>
          </div>

          <div class="card p-3 bg-light border mb-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="fw-bold text-secondary m-0"><i class="fa-solid fa-clock-rotate-left me-2"></i>Talaan kung kelan nagdadagdag ng Produkto (Stock-In History & Supplier)</h5>
            </div>
            <div class="input-group mb-3">
              <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
              <input type="text" id="searchStockInInput" class="form-control" placeholder="I-search ang pangalan ng produkto, supplier o petsa..." oninput="renderStockInHistory()">
            </div>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Petsa (Date Added)</th>
                    <th>Product Name</th>
                    <th>Description</th>
                    <th class="text-center">Ibinagdag (Stock In Qty)</th>
                    <th>Supplier / Galing Kay</th>
                    <th>Uri / Note</th>
                  </tr>
                </thead>
                <tbody id="stockInHistoryBody">
                  <!-- Dynamic Content -->
                </tbody>
              </table>
            </div>
          </div>

          <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-users me-2"></i>Listahan ng mga Bumili ng Item (Customer Purchases)</h5>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle bg-white">
              <thead class="table-light">
                <tr>
                  <th>Date</th>
                  <th>Customer Name</th>
                  <th>Location</th>
                  <th>Product Name</th>
                  <th>Description</th>
                  <th class="text-center">Quantity (Ilan)</th>
                  <th class="text-end">Cost (₱)</th>
                  <th class="text-end">Price (₱)</th>
                  <th class="text-end">Total Amount (₱)</th>
                </tr>
              </thead>
              <tbody id="customerSalesLogBody">
                <!-- Dynamic Content -->
              </tbody>
            </table>
          </div>

        </div>
      </div>

      <!-- ================= 5. SALARY & EXPENSES STANDALONE LEDGER TAB ================= -->
      <div class="tab-pane fade" id="expenses-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-receipt me-2"></i>Salary & Expenses Ledger (Araw-arawang Nagastos)</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-outline-secondary" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Expenses Ledger
              </button>
              <button class="btn btn-danger fw-bold" onclick="addStandaloneExpenseRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Expense / Salary Line
              </button>
             
              <!-- PER-DAY / PER-MONTH FILTER TOGGLE -->
              <div class="d-flex align-items-center gap-1 ms-2">
                <label class="fw-bold small text-nowrap">View:</label>
                <select id="expenseViewMode" class="form-select form-select-sm" onchange="toggleExpenseViewMode()">
                  <option value="month">Buwanan (Month)</option>
                  <option value="day" selected>Pangkalahatang Araw (Per Day)</option>
                </select>
                <input type="month" id="standaloneExpenseMonth" class="form-control form-control-sm" style="display: none;" onchange="renderStandaloneExpensesLedger()">
                <input type="date" id="standaloneExpenseDate" class="form-control form-control-sm" onchange="renderStandaloneExpensesLedger()">
              </div>
            </div>
          </div>

          <!-- Summary Cards para sa Expenses & Salary -->
          <div class="row g-3 mb-4">
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-danger border-4">
                <span class="text-muted small fw-bold">TOTAL SALARY (Sahod)</span>
                <h4 class="text-danger mt-1 mb-0" id="totalSalarySumDisplay">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-warning border-4">
                <span class="text-muted small fw-bold">TOTAL EXPENSES (Gastos)</span>
                <h4 class="text-warning-emphasis mt-1 mb-0" id="totalExpenseSumDisplay">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-dark border-4">
                <span class="text-muted small fw-bold">KABUUANG CASH OUT (Salary + Expenses)</span>
                <h4 class="text-dark fw-bold mt-1 mb-0" id="totalCombinedExpenseDisplay">₱0.00</h4>
              </div>
            </div>
          </div>

          <div class="input-group mb-3">
            <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
            <input type="text" id="searchStandaloneExpenseInput" class="form-control" placeholder="I-search ang petsa, pangalan ng sahod o expenses..." oninput="renderStandaloneExpensesLedger()">
          </div>

          <div class="table-responsive">
            <table class="table table-bordered align-middle bg-white">
              <thead class="table-dark">
                <tr>
                  <th style="width: 150px;">Date (Petsa)</th>
                  <th>Salary Name / Description</th>
                  <th style="width: 170px;" class="text-end">Salary Amount (₱)</th>
                  <th>Expenses Name / Description</th>
                  <th style="width: 170px;" class="text-end">Expenses Amount (₱)</th>
                  <th class="col-action no-print text-center"><i class="fa-solid fa-trash"></i></th>
                </tr>
              </thead>
              <tbody id="standaloneExpenseTableBody">
                <!-- Dynamic Ledger Rows -->
              </tbody>
              <tfoot class="table-secondary fw-bold" id="standaloneExpenseTableFooter">
                <!-- Subtotals -->
              </tfoot>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 6. D/ECO BOSS STANDALONE LEDGER TAB ================= -->
      <div class="tab-pane fade" id="boss-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Ledger (Capital & Withdrawals)</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-outline-secondary" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Boss Ledger
              </button>
              <button class="btn btn-warning fw-bold text-dark" data-bs-toggle="modal" data-bs-target="#bossModal">
                <i class="fa-solid fa-plus me-1"></i> Add Boss Adjustment
              </button>
             
              <!-- PER-DAY FILTER TOGGLE / DATE SELECTOR SA BOSS LEDGER -->
              <div class="d-flex align-items-center gap-1 ms-2">
                <label class="fw-bold small text-nowrap">View:</label>
                <select id="bossViewMode" class="form-select form-select-sm" onchange="toggleBossViewMode()">
                  <option value="month">Buwanan (Month)</option>
                  <option value="day">Pangkalahatang Araw (Per Day)</option>
                </select>
                <input type="month" id="standaloneBossMonth" class="form-control form-control-sm" onchange="renderStandaloneBossLedger()">
                <input type="date" id="standaloneBossDate" class="form-control form-control-sm" style="display: none;" onchange="renderStandaloneBossLedger()">
              </div>
            </div>
          </div>

          <!-- Summary Cards para sa D/Eco Boss -->
          <div class="row g-3 mb-4">
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-success border-4">
                <span class="text-muted small fw-bold">TOTAL BOSS ADDITIONS (Capital In)</span>
                <h4 class="text-success mt-1 mb-0" id="totalBossAddDisplay">+₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-danger border-4">
                <span class="text-muted small fw-bold">TOTAL BOSS WITHDRAWALS (Cash Out)</span>
                <h4 class="text-danger mt-1 mb-0" id="totalBossSubDisplay">-₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light border-start border-primary border-4">
                <span class="text-muted small fw-bold">NET BOSS BALANCE ADJUSTMENT</span>
                <h4 class="text-primary fw-bold mt-1 mb-0" id="totalBossNetDisplay">₱0.00</h4>
              </div>
            </div>
          </div>

          <div class="input-group mb-3">
            <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
            <input type="text" id="searchStandaloneBossInput" class="form-control" placeholder="I-search ang petsa, uri o notes sa D/Eco Boss ledger..." oninput="renderStandaloneBossLedger()">
          </div>

          <div class="table-responsive">
            <table class="table table-bordered align-middle bg-white">
              <thead class="table-dark">
                <tr>
                  <th style="width: 150px;">Date</th>
                  <th>Description / Type</th>
                  <th style="width: 180px;" class="text-end">Amount (₱)</th>
                  <th class="text-center no-print" style="width: 120px;">Actions</th>
                </tr>
              </thead>
              <tbody id="standaloneBossTableBody">
                <!-- Dynamic Boss Ledger Rows -->
              </tbody>
              <tfoot class="table-secondary fw-bold" id="standaloneBossTableFooter">
                <!-- Subtotal -->
              </tfoot>
            </table>
          </div>
        </div>
      </div>

      <!-- ================= 7. MONTHLY AUDIT TAB ================= -->
      <div class="tab-pane fade" id="audit-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-chart-pie me-2"></i>Monthly Audit & Net Profit Computation</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-outline-secondary" onclick="window.print()">
                <i class="fa-solid fa-print me-1"></i> Print Audit Report
              </button>
              <button class="btn btn-warning fw-bold text-dark me-2" data-bs-toggle="modal" data-bs-target="#bossModal">
                <i class="fa-solid fa-user-tie me-1"></i> Add D/Eco Boss Adjustment
              </button>
              <label class="fw-bold me-1">Filter Month:</label>
              <input type="month" id="auditMonth" class="form-control" onchange="generateMonthlyAudit()">
            </div>
          </div>

          <!-- GLOBAL SEARCH LAHAT NG TRANSAKSIYON -->
          <div class="card p-3 bg-white border mb-4 shadow-sm">
            <h6 class="fw-bold text-primary mb-2"><i class="fa-solid fa-magnifying-glass me-2"></i>Search All Transactions & Logs (POS, Daily, Utang, Inventory)</h6>
            <div class="row g-2">
              <div class="col-md-9">
                <input type="text" id="globalSearchInput" class="form-control" placeholder="I-type ang pangalan ng customer, produkto, o petsa (e.g. 2026-06-06 o Juan)..." oninput="renderGlobalSearchResults()">
              </div>
              <div class="col-md-3">
                <button class="btn btn-outline-primary w-100 fw-semibold" onclick="clearGlobalSearch()"><i class="fa-solid fa-rotate-right me-1"></i> Reset Search</button>
              </div>
            </div>
            <div id="globalSearchResultsWrapper" class="mt-3" style="display: none;">
              <div class="table-responsive">
                <table class="table table-bordered table-sm table-hover align-middle">
                  <thead class="table-dark">
                    <tr>
                      <th>Date</th>
                      <th>Customer / Ref</th>
                      <th>Location</th>
                      <th>Products / Details</th>
                      <th>Total (₱)</th>
                      <th>Paid (₱)</th>
                      <th>Balance</th>
                      <th>Status / Action</th>
                    </tr>
                  </thead>
                  <tbody id="globalSearchResultsBody">
                    <!-- Dynamic Search Results -->
                  </tbody>
                </table>
              </div>
            </div>
          </div>

          <!-- HIWAY & BYAHE SUMMARY CARDS FOR MONTHLY AUDIT -->
          <div class="row g-3 mb-3">
            <div class="col-md-6">
              <div class="card p-3 stat-card bg-light border-start border-primary border-4">
                <span class="text-muted small fw-bold">HIWAY MONTHLY TOTALS</span>
                <div class="d-flex justify-content-between mt-2">
                  <div>
                    <small class="text-muted d-block">Sales:</small>
                    <h5 class="text-primary mb-0" id="auditHiwaySales">₱0.00</h5>
                  </div>
                  <div>
                    <small class="text-muted d-block">Cost (Puhunan):</small>
                    <h5 class="text-secondary mb-0" id="auditHiwayCost">₱0.00</h5>
                  </div>
                  <div>
                    <small class="text-muted d-block">Hiway Net:</small>
                    <h5 class="text-success fw-bold mb-0" id="auditHiwayNetProfit">₱0.00</h5>
                  </div>
                </div>
              </div>
            </div>
            <div class="col-md-6">
              <div class="card p-3 stat-card bg-light border-start border-info border-4">
                <span class="text-muted small fw-bold">BYAHE MONTHLY TOTALS</span>
                <div class="d-flex justify-content-between mt-2">
                  <div>
                    <small class="text-muted d-block">Sales:</small>
                    <h5 class="text-primary mb-0" id="auditByaheSales">₱0.00</h5>
                  </div>
                  <div>
                    <small class="text-muted d-block">Cost (Puhunan):</small>
                    <h5 class="text-secondary mb-0" id="auditByaheCost">₱0.00</h5>
                  </div>
                  <div>
                    <small class="text-muted d-block">Byahe Net:</small>
                    <h5 class="text-success fw-bold mb-0" id="auditByaheNetProfit">₱0.00</h5>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="row g-3 mb-4">
            <div class="col-md-2">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">GROSS SALES</span>
                <h5 class="text-primary mt-1 mb-0" id="auditTotalSales">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-2">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #6c757d;">
                <span class="text-muted small fw-bold">TOTAL COGS (Puhunan)</span>
                <h5 class="text-secondary mt-1 mb-0" id="auditTotalCost">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-2">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;">
                <span class="text-muted small fw-bold">SUBTOTAL NET (Hiway + Byahe)</span>
                <h5 class="text-teal mt-1 mb-0" id="auditGrossProfit" style="color: #00897b;">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #c62828;">
                <span class="text-muted small fw-bold">EXPENSES / SALARIES</span>
                <h5 class="text-danger mt-1 mb-0" id="auditExpenses">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">FINAL NET PROFIT (Grand Net)</span>
                <h4 class="text-success fw-bold mt-1 mb-0" id="auditNetProfit">₱0.00</h4>
              </div>
            </div>
          </div>

          <!-- DAILY NET PROFIT BREAKDOWN TABLE IN AUDIT -->
          <div class="card p-3 bg-light mb-4 border">
            <div class="d-flex justify-content-between align-items-center mb-2">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-calendar-days me-2"></i>Daily Net Profit Breakdown (Araw-arawang Kita) - Pindutin ang "View" para balikan ang Daily Report</h6>
            </div>
            <div class="table-responsive">
              <table class="table table-bordered table-sm align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Date</th>
                    <th class="text-end">Hiway Net (₱)</th>
                    <th class="text-end">Byahe Net (₱)</th>
                    <th class="text-end">Subtotal Net (₱)</th>
                    <th class="text-end">Final Net Profit (₱)</th>
                    <th class="text-center no-print" style="width: 140px;">Action / Balikan</th>
                  </tr>
                </thead>
                <tbody id="auditDailyBreakdownBody">
                  <!-- Dynamic Daily Breakdown Rows -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- SALARY & EXPENSES BREAKDOWN -->
          <div class="card p-3 bg-light mb-4 border">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-receipt me-2"></i>Itemized Salary & Expenses Breakdown</h6>
              <button class="btn btn-sm btn-outline-danger" onclick="addExpenseRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Expense Line
              </button>
            </div>
           
            <div class="input-group mb-3">
              <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
              <input type="text" id="searchExpenseInput" class="form-control" placeholder="I-search ang pangalan ng Salary o Expense o petsa..." oninput="renderExpensesTable()">
            </div>

            <div class="table-responsive">
              <table class="table table-bordered align-middle bg-white" id="expenseTable">
                <thead class="table-light">
                  <tr>
                    <th style="width: 180px;">Date (Petsa)</th>
                    <th>Salary Name / Description</th>
                    <th style="width: 180px;">Salary Amount (₱)</th>
                    <th>Expenses Name / Description</th>
                    <th style="width: 180px;">Expenses Amount (₱)</th>
                    <th class="col-action no-print"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="expenseTableBody">
                  <!-- Dynamic Expense Rows -->
                </tbody>
                <tfoot class="table-secondary fw-bold" id="expenseTableFooter">
                  <!-- Dynamic Subtotal / Total Row -->
                </tfoot>
              </table>
            </div>
          </div>

          <!-- D/ECO BOSS TRANSACTIONS LOG -->
          <div class="card p-3 bg-light mb-4 border">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-user-tie me-2"></i>D/Eco Boss Transactions Log</h6>
              <button class="btn btn-sm btn-outline-warning text-dark fw-bold" data-bs-toggle="modal" data-bs-target="#bossModal">
                <i class="fa-solid fa-plus me-1"></i> Add Boss Adjustment
              </button>
            </div>
           
            <div class="input-group mb-3">
              <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
              <input type="text" id="searchBossInput" class="form-control" placeholder="I-search ang petsa, uri o notes sa D/Eco Boss log..." oninput="generateMonthlyAudit()">
            </div>

            <div class="table-responsive">
              <table class="table table-sm table-bordered bg-white align-middle">
                <thead class="table-light">
                  <tr>
                    <th style="width: 150px;">Date</th>
                    <th>Description / Type</th>
                    <th style="width: 180px;" class="text-end">Amount (₱)</th>
                    <th class="text-center no-print" style="width: 120px;">Actions</th>
                  </tr>
                </thead>
                <tbody id="bossLogsBody">
                  <!-- Dynamic Content -->
                </tbody>
                <tfoot class="table-secondary fw-bold" id="bossLogsFooter">
                  <!-- Subtotal row rendered dynamically -->
                </tfoot>
              </table>
            </div>
          </div>

        </div>
      </div>

    </div>
  </div>

  <!-- MODAL: STANDALONE MANUAL PAYMENT / COLLECTION -->
  <div class="modal fade" id="standalonePaymentModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-success text-white">
          <h5 class="modal-title"><i class="fa-solid fa-hand-holding-dollar me-2"></i>Add Manual Payment / Collection</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="standalonePaymentForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date (Petsa ng Bayad):</label>
              <input type="date" id="stdPayDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Customer Name:</label>
              <input type="text" id="stdPayCustomer" class="form-control" placeholder="e.g. Juan Dela Cruz" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Amount Paid (₱):</label>
              <input type="number" step="0.01" min="0.01" id="stdPayAmount" class="form-control" placeholder="0.00" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Payment Method:</label>
              <select id="stdPayMethod" class="form-select" required>
                <option value="Cash">Cash</option>
                <option value="Byahe Cash">Byahe Cash</option>
                <option value="GCash">GCash</option>
                <option value="Bank Transfer">Bank Transfer (BT)</option>
                <option value="Cheque">Cheque</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Notes / Remarks (Optional):</label>
              <input type="text" id="stdPayNotes" class="form-control" placeholder="e.g., Bayad sa lumang utang na hindi naka-encode">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk me-1"></i>Save Collection</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: PAYMENT / BAYAD SA UTANG -->
  <div class="modal fade" id="paymentModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-success text-white">
          <h5 class="modal-title"><i class="fa-solid fa-peso-sign me-2"></i>Magbayad ng Utang (Payment)</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="paymentForm">
          <div class="modal-body">
            <input type="hidden" id="payTxId">
            <div class="mb-3">
              <label class="form-label fw-semibold">Customer Name:</label>
              <input type="text" id="payCustomerName" class="form-control bg-light" readonly>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="text" id="payTotalAmount" class="form-control bg-light" readonly>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Remaining Balance (₱):</label>
                <input type="text" id="payRemainingBalance" class="form-control bg-light text-danger fw-bold" readonly>
              </div>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Amount to Pay Now (₱):</label>
              <input type="number" step="0.01" min="0.01" id="payAmountNow" class="form-control" required placeholder="0.00">
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Payment Method:</label>
              <select id="payMethod" class="form-select" required>
                <option value="Cash">Cash</option>
                <option value="Byahe Cash">Byahe Cash</option>
                <option value="GCash">GCash</option>
                <option value="Bank Transfer">Bank Transfer (BT)</option>
                <option value="Cheque">Cheque</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Date of Payment:</label>
              <input type="date" id="payDate" class="form-control" required>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-check me-1"></i>I-save ang Bayad</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: EDIT TRANSACTION -->
  <div class="modal fade" id="editTransactionModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-pen-to-square me-2"></i>Edit Transaction & Cost</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="editTransactionForm">
          <div class="modal-body">
            <input type="hidden" id="editTxId">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date (Petsa):</label>
              <input type="date" id="editTxDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Customer Name:</label>
              <input type="text" id="editCustomerName" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Location / Uri:</label>
              <select id="editLocation" class="form-select" required>
                <option value="Hiway">Hiway</option>
                <option value="Byahe">Byahe</option>
                <option value="Inventory Only">Inventory Only</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Products Bought:</label>
              <input type="text" id="editProduct" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Container Status:</label>
              <input type="text" id="editContainerInfo" class="form-control">
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Total Cost / Puhunan (₱):</label>
                <input type="number" step="0.01" id="editTotalCost" class="form-control" required>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="number" step="0.01" id="editTotal" class="form-control" required oninput="calculateEditBalance()">
              </div>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Paid Amount (₱):</label>
                <input type="number" step="0.01" id="editPaid" class="form-control" required oninput="calculateEditBalance()">
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Remaining Balance (₱):</label>
                <input type="number" step="0.01" id="editBalance" class="form-control bg-light" readonly>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk me-1"></i>Update Transaction</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: CHANGE PROFILE -->
  <div class="modal fade" id="changeProfileModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-id-card me-2"></i>Edit My Profile</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="changeProfileForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Display Name:</label>
              <input type="text" id="profDisplayName" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">New Password:</label>
              <input type="password" id="profPassword" class="form-control" placeholder="Iwanang blangko kung ayaw palitan">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk me-1"></i>Save Changes</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: USER MANAGEMENT -->
  <div class="modal fade" id="userManagementModal" tabindex="-1">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <div class="modal-header bg-dark text-white">
          <h5 class="modal-title"><i class="fa-solid fa-users-gear me-2"></i>User & Admin Accounts Management</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <h6 class="fw-bold mb-3 text-primary"><i class="fa-solid fa-user-plus me-1"></i>Add New System User / Admin</h6>
          <form id="newUserForm" class="row g-2 mb-4 bg-light p-3 border rounded">
            <div class="col-md-3">
              <input type="text" id="newAccName" class="form-control form-control-sm" placeholder="Full Name" required>
            </div>
            <div class="col-md-3">
              <input type="text" id="newAccUser" class="form-control form-control-sm" placeholder="Username" required>
            </div>
            <div class="col-md-3">
              <input type="password" id="newAccPass" class="form-control form-control-sm" placeholder="Password" required>
            </div>
            <div class="col-md-2">
              <select id="newAccRole" class="form-select form-select-sm">
                <option value="Staff">Staff</option>
                <option value="Admin">Admin</option>
              </select>
            </div>
            <div class="col-md-1">
              <button type="submit" class="btn btn-sm btn-success w-100"><i class="fa-solid fa-plus"></i></button>
            </div>
          </form>

          <h6 class="fw-bold mb-2"><i class="fa-solid fa-users me-1"></i>Existing System Users</h6>
          <div class="table-responsive">
            <table class="table table-bordered align-middle table-sm">
              <thead class="table-light">
                <tr>
                  <th>Name</th>
                  <th>Username</th>
                  <th>Role</th>
                  <th class="text-center">Action</th>
                </tr>
              </thead>
              <tbody id="userListBody">
                <!-- User rows rendered dynamically -->
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
                <option value="ADD">Boss Addition / Capital Cash In (+ Subtotal Net)</option>
                <option value="SUB">Boss Withdrawal / Cash Out (- Subtotal Net)</option>
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
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</label>
            <button type="submit" class="btn btn-dark"><i class="fa-solid fa-floppy-disk me-1"></i>Save Adjustment</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: EDIT D/ECO BOSS ADJUSTMENT -->
  <div class="modal fade" id="editBossModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-warning">
          <h5 class="modal-title fw-bold text-dark"><i class="fa-solid fa-pen-to-square me-2"></i>Edit D/Eco Boss Adjustment</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <form id="editBossForm">
          <div class="modal-body">
            <input type="hidden" id="editBossIndex">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date:</label>
              <input type="date" id="editBossDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Adjustment Type:</label>
              <select id="editBossType" class="form-select">
                <option value="ADD">Boss Addition / Capital Cash In (+ Subtotal Net)</option>
                <option value="SUB">Boss Withdrawal / Cash Out (- Subtotal Net)</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Amount (₱):</label>
              <input type="number" step="0.01" id="editBossAmount" class="form-control" placeholder="0.00" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Notes / Description:</label>
              <input type="text" id="editBossNotes" class="form-control" placeholder="e.g., Personal Withdrawal, Additional Capital">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-warning fw-bold text-dark"><i class="fa-solid fa-floppy-disk me-1"></i>Update Adjustment</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: ADD PRODUCT -->
  <div class="modal fade" id="addProductModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-success text-white">
          <h5 class="modal-title"><i class="fa-solid fa-plus-circle me-2"></i>Add New Product Master</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="addProductForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Category:</label>
              <select id="newProdCategory" class="form-select" required>
                <option value="palm_coco">Palm & Coco</option>
                <option value="dedicated">Dedicated Products</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name:</label>
              <input type="text" id="newProdName" class="form-control" placeholder="e.g. Sugar, Oil, Bigas" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Description (Uri/Brand):</label>
              <input type="text" id="newProdDesc" class="form-control" placeholder="e.g. Premium, Class A, Local" required>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Cost / Unit (₱):</label>
                <input type="number" step="0.01" min="0" id="newProdCost" class="form-control" placeholder="0.00" required>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Price / Unit (₱):</label>
                <input type="number" step="0.01" min="0" id="newProdPrice" class="form-control" placeholder="0.00" required>
              </div>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Beginning Stock:</label>
              <input type="number" step="any" min="0" id="newProdStock" class="form-control" value="0" required>
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

  <!-- MODAL: RETURN / ISAULI ITEM -->
  <div class="modal fade" id="returnModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-info text-white">
          <h5 class="modal-title"><i class="fa-solid fa-rotate-left me-2"></i>Item Return / Isauli ang Produkto</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="returnForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date of Return (Petsa):</label>
              <input type="date" id="returnDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Customer / Reference Name:</label>
              <input type="text" id="returnCustomer" class="form-control" placeholder="e.g. Juan Dela Cruz" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Category:</label>
              <select id="returnCategory" class="form-select" required onchange="populateReturnProducts()">
                <option value="palm_coco">Palm & Coco</option>
                <option value="dedicated">Dedicated Products</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Select Product:</label>
              <select id="returnProductSelect" class="form-select" required>
                <!-- Dynamic Products -->
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Quantity to Return (Ilang Isauli):</label>
              <input type="number" step="any" min="0.5" id="returnQty" class="form-control" value="1" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Reason / Notes:</label>
              <input type="text" id="returnNotes" class="form-control" placeholder="e.g., Sobra ang kuha, nasira, etc.">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-info text-white fw-bold"><i class="fa-solid fa-check me-1"></i>Process Return</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- Bootstrap JS Bundle CDN -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>

  <!-- JavaScript Code na may Product + Description Unique Key para sa Inventory -->
  <script>
    // Global App State & Default Master Inventory Lists
    let currentUser = null;
    let usersList = JSON.parse(localStorage.getItem('rmv_users')) || [
      { name: 'Admin User', username: 'admin', password: '123', role: 'Admin' },
      { name: 'Staff User', username: 'staff', password: '123', role: 'Staff' }
    ];

    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let standalonePayments = JSON.parse(localStorage.getItem('rmv_standalone_payments')) || [];
    let standaloneExpenses = JSON.parse(localStorage.getItem('rmv_standalone_expenses')) || [];
    let bossAdjustments = JSON.parse(localStorage.getItem('rmv_boss_adjustments')) || [];
    let stockInHistory = JSON.parse(localStorage.getItem('rmv_stock_in_history')) || [];
    let itemReturns = JSON.parse(localStorage.getItem('rmv_item_returns')) || [];

    let palmCocoMaster = JSON.parse(localStorage.getItem('rmv_palm_coco_master')) || [
      { name: "Palm Oil", description: "Standard", cost: 800, price: 900, stock: 50 },
      { name: "Coco Oil", description: "Pure", cost: 850, price: 950, stock: 40 }
    ];

    let dedicatedMaster = JSON.parse(localStorage.getItem('rmv_dedicated_master')) || [
      { name: "VMC White", description: "50kg", cost: 2200, price: 2350, stock: 30 },
      { name: "Busco", description: "50kg", cost: 2150, price: 2300, stock: 25 },
      { name: "Harina", description: "Class A", cost: 1100, price: 1250, stock: 20 }
    ];

    // Helper: Bumuo ng Unique Key gamit ang Pangalan at Description para hiwalay ang imbentaryo
    function getProductKey(name, desc) {
      return (name ? name.trim().toLowerCase() : '') + '___' + (desc ? desc.trim().toLowerCase() : '');
    }

    window.addEventListener('DOMContentLoaded', () => {
      // Set default dates to today
      const todayStr = new Date().toISOString().split('T')[0];
      if(document.getElementById('saleDate')) document.getElementById('saleDate').value = todayStr;
      if(document.getElementById('dailyReportDate')) document.getElementById('dailyReportDate').value = todayStr;
      if(document.getElementById('inventorySheetDate')) document.getElementById('inventorySheetDate').value = todayStr;
      if(document.getElementById('auditMonth')) document.getElementById('auditMonth').value = todayStr.substring(0, 7);
      if(document.getElementById('standaloneExpenseDate')) document.getElementById('standaloneExpenseDate').value = todayStr;
      if(document.getElementById('standaloneExpenseMonth')) document.getElementById('standaloneExpenseMonth').value = todayStr.substring(0, 7);
      if(document.getElementById('standaloneBossDate')) document.getElementById('standaloneBossDate').value = todayStr;
      if(document.getElementById('standaloneBossMonth')) document.getElementById('standaloneBossMonth').value = todayStr.substring(0, 7);
      if(document.getElementById('dueDate')) document.getElementById('dueDate').value = todayStr;
      if(document.getElementById('stdPayDate')) document.getElementById('stdPayDate').value = todayStr;
      if(document.getElementById('bossDate')) document.getElementById('bossDate').value = todayStr;
      if(document.getElementById('returnDate')) document.getElementById('returnDate').value = todayStr;

      checkLoginStatus();
      addPosRow();
      renderInventoryTables();
    });

    // Login Authentication Functions
    document.getElementById('loginForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const u = document.getElementById('loginUsername').value.trim();
      const p = document.getElementById('loginPassword').value.trim();
      const found = usersList.find(x => x.username === u && x.password === p);
      if(found) {
        currentUser = found;
        localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
        document.getElementById('loginOverlay').style.display = 'none';
        checkLoginStatus();
      } else {
        document.getElementById('loginError').classList.remove('d-none');
      }
    });

    function checkLoginStatus() {
      const savedUser = localStorage.getItem('rmv_current_user');
      if(savedUser) {
        currentUser = JSON.parse(savedUser);
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('currentUserName').innerText = currentUser.name + ' (' + currentUser.role + ')';
        if(currentUser.role !== 'Admin') {
          document.querySelectorAll('.admin-only').forEach(el => el.style.display = 'none');
        } else {
          document.querySelectorAll('.admin-only').forEach(el => el.style.display = '');
        }
      } else {
        document.getElementById('loginOverlay').style.display = 'flex';
      }
    }

    function logout() {
      localStorage.removeItem('rmv_current_user');
      currentUser = null;
      location.reload();
    }

    function manualSaveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_standalone_payments', JSON.stringify(standalonePayments));
      localStorage.setItem('rmv_standalone_expenses', JSON.stringify(standaloneExpenses));
      localStorage.setItem('rmv_boss_adjustments', JSON.stringify(bossAdjustments));
      localStorage.setItem('rmv_stock_in_history', JSON.stringify(stockInHistory));
      localStorage.setItem('rmv_item_returns', JSON.stringify(itemReturns));
      localStorage.setItem('rmv_palm_coco_master', JSON.stringify(palmCocoMaster));
      localStorage.setItem('rmv_dedicated_master', JSON.stringify(dedicatedMaster));
      alert('Nai-save na ang lahat ng datos nang matagumpay!');
    }

    function exportDataBackup() {
      const dataObj = {
        transactions, standalonePayments, standaloneExpenses, bossAdjustments,
        stockInHistory, itemReturns, palmCocoMaster, dedicatedMaster, usersList
      };
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(dataObj, null, 2));
      const dlAnchor = document.createElement('a');
      dlAnchor.setAttribute("href", dataStr);
      dlAnchor.setAttribute("download", "RMVillasis_Backup_" + new Date().toISOString().split('T')[0] + ".json");
      document.body.appendChild(dlAnchor);
      dlAnchor.click();
      dlAnchor.remove();
    }

    function importDataBackup(event) {
      const fileReader = new FileReader();
      if(event.target.files[0]) {
        fileReader.readAsText(event.target.files[0], "UTF-8");
        fileReader.onload = function(e) {
          try {
            const imported = JSON.parse(e.target.result);
            if(imported.transactions) transactions = imported.transactions;
            if(imported.standalonePayments) standalonePayments = imported.standalonePayments;
            if(imported.standaloneExpenses) standaloneExpenses = imported.standaloneExpenses;
            if(imported.bossAdjustments) bossAdjustments = imported.bossAdjustments;
            if(imported.stockInHistory) stockInHistory = imported.stockInHistory;
            if(imported.itemReturns) itemReturns = imported.itemReturns;
            if(imported.palmCocoMaster) palmCocoMaster = imported.palmCocoMaster;
            if(imported.dedicatedMaster) dedicatedMaster = imported.dedicatedMaster;
            if(imported.usersList) usersList = imported.usersList;
            manualSaveData();
            alert('Naibalik (Restored) na ang mga datos mula sa backup file!');
            location.reload();
          } catch(err) {
            alert('Mali ang format ng file o may sira ang JSON.');
          }
        };
      }
    }

    // POS Dynamic Rows & Calculations
    function addPosRow() {
      const tbody = document.getElementById('posItemsBody');
      const tr = document.createElement('tr');
      const isInvOnly = document.getElementById('inventoryOnlyMode').checked;
      const priceDisplay = isInvOnly ? 'display: none;' : '';

      tr.innerHTML = `
        <td><input type="text" class="form-control form-control-sm pos-name" placeholder="Pangalan ng Produkto" required oninput="autoCompleteProduct(this)"></td>
        <td><input type="text" class="form-control form-control-sm pos-desc" placeholder="Uri / Description / Brand"></td>
        <td><input type="number" step="any" min="0.5" class="form-control form-control-sm pos-qty text-center" value="1" required oninput="calculateTotal()"></td>
        <td style="${priceDisplay}"><input type="number" step="0.01" min="0" class="form-control form-control-sm pos-cost text-end" placeholder="0.00" oninput="calculateTotal()"></td>
        <td style="${priceDisplay}"><input type="number" step="0.01" min="0" class="form-control form-control-sm pos-price text-end" placeholder="0.00" oninput="calculateTotal()"></td>
        <td style="${priceDisplay}"><input type="text" class="form-control form-control-sm bg-light pos-subtotal text-end" readonly value="0.00"></td>
        <td class="col-action"><button type="button" class="btn btn-sm btn-outline-danger" onclick="removePosRow(this)"><i class="fa-solid fa-trash"></i></button></td>
      `;
      tbody.appendChild(tr);
      calculateTotal();
    }

    function removePosRow(btn) {
      if(document.querySelectorAll('#posItemsBody tr').length > 1) {
        btn.closest('tr').remove();
        calculateTotal();
      } else {
        alert('Kailangang may kahit isang item sa transaksyon.');
      }
    }

    function toggleInventoryOnlyMode() {
      const isInv = document.getElementById('inventoryOnlyMode').checked;
      const priceCols = document.querySelectorAll('.price-col');
      priceCols.forEach(col => col.style.display = isInv ? 'none' : '');
      document.getElementById('financialSectionBox').style.display = isInv ? 'none' : '';
      document.getElementById('containerSectionBox').style.display = isInv ? 'none' : '';
      if(isInv) {
        document.getElementById('transactionLocation').value = 'Inventory Only';
      } else {
        document.getElementById('transactionLocation').value = 'Hiway';
      }
      document.querySelectorAll('#posItemsBody tr').forEach(tr => {
        const costCell = tr.querySelectorAll('td')[3];
        const priceCell = tr.querySelectorAll('td')[4];
        const subCell = tr.querySelectorAll('td')[5];
        if(costCell) costCell.style.display = isInv ? 'none' : '';
        if(priceCell) priceCell.style.display = isInv ? 'none' : '';
        if(subCell) subCell.style.display = isInv ? 'none' : '';
      });
      calculateTotal();
    }

    function toggleContainerFields() {
      const status = document.getElementById('containerStatus').value;
      const qtyGroup = document.querySelector('.container-qty-group');
      const depGroup = document.querySelector('.container-deposit-group');
      if(status === 'DEPOSIT') {
        qtyGroup.style.display = '';
        depGroup.style.display = '';
      } else if(status === 'HIRAM') {
        qtyGroup.style.display = '';
        depGroup.style.display = 'none';
      } else {
        qtyGroup.style.display = 'none';
        depGroup.style.display = 'none';
      }
      calculateTotal();
    }

    function toggleCreditFields() {
      const pType = document.getElementById('paymentType').value;
      const credSec = document.getElementById('creditFieldsSection');
      const amtPaid = document.getElementById('amountPaidNow');
      const totalAmt = parseFloat(document.getElementById('totalAmount').value) || 0;

      if(pType === 'CREDIT') {
        credSec.style.display = 'block';
        amtPaid.value = '0.00';
      } else if(pType === 'PARTIAL') {
        credSec.style.display = 'block';
      } else {
        credSec.style.display = 'none';
        amtPaid.value = totalAmt.toFixed(2);
      }
      calculateBalance();
    }

    function togglePaymentMethodFields() {
      const method = document.getElementById('paymentMethod').value;
      const multiBox = document.getElementById('multiPaymentFields');
      if(method === 'Multi-Payment') {
        multiBox.style.display = 'block';
        const totalAmt = parseFloat(document.getElementById('totalAmount').value) || 0;
        document.getElementById('multiCashAmt').value = totalAmt.toFixed(2);
      } else {
        multiBox.style.display = 'none';
      }
    }

    function calculateTotal() {
      let sum = 0;
      const isInv = document.getElementById('inventoryOnlyMode').checked;
      
      document.querySelectorAll('#posItemsBody tr').forEach(tr => {
        const qty = parseFloat(tr.querySelector('.pos-qty').value) || 0;
        const price = parseFloat(tr.querySelector('.pos-price') ? tr.querySelector('.pos-price').value : 0) || 0;
        const sub = qty * price;
        if(tr.querySelector('.pos-subtotal')) tr.querySelector('.pos-subtotal').value = sub.toFixed(2);
        sum += sub;
      });

      if(!isInv) {
        const contStatus = document.getElementById('containerStatus').value;
        if(contStatus === 'DEPOSIT') {
          const cQty = parseFloat(document.getElementById('containerQty').value) || 0;
          const cRate = parseFloat(document.getElementById('containerDepositRate').value) || 0;
          sum += (cQty * cRate);
        }
      }

      document.getElementById('totalAmount').value = sum.toFixed(2);
      const pType = document.getElementById('paymentType').value;
      if(pType === 'FULL' || isInv) {
        document.getElementById('amountPaidNow').value = sum.toFixed(2);
      }
      calculateBalance();
    }

    function calculateBalance() {
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
      const bal = total - paid;
      document.getElementById('remainingBalance').value = bal > 0 ? bal.toFixed(2) : '0.00';
    }

    function autoCompleteProduct(input) {
      const val = input.value.trim().toLowerCase();
      const tr = input.closest('tr');
      const descInput = tr.querySelector('.pos-desc');
      const costInput = tr.querySelector('.pos-cost');
      const priceInput = tr.querySelector('.pos-price');

      // Hanapin sa Master lists gamit ang Pangalan
      let found = palmCocoMaster.find(x => x.name.toLowerCase() === val);
      if(!found) found = dedicatedMaster.find(x => x.name.toLowerCase() === val);

      if(found) {
        if(descInput && !descInput.value) descInput.value = found.description || '';
        if(costInput && !costInput.value) costInput.value = found.cost || 0;
        if(priceInput && !priceInput.value) priceInput.value = found.price || 0;
        calculateTotal();
      }
    }

    // POS Form Submission & Inventory Stock Update gamit ang Product+Description Unique Key
    document.getElementById('posForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('saleDate').value;
      const customer = document.getElementById('customerName').value.trim();
      const location = document.getElementById('transactionLocation').value;
      const isInv = document.getElementById('inventoryOnlyMode').checked;

      let items = [];
      let totalCost = 0;

      document.querySelectorAll('#posItemsBody tr').forEach(tr => {
        const name = tr.querySelector('.pos-name').value.trim();
        const desc = tr.querySelector('.pos-desc').value.trim();
        const qty = parseFloat(tr.querySelector('.pos-qty').value) || 0;
        const cost = isInv ? 0 : (parseFloat(tr.querySelector('.pos-cost').value) || 0);
        const price = isInv ? 0 : (parseFloat(tr.querySelector('.pos-price').value) || 0);
        const subtotal = qty * price;
        totalCost += (qty * cost);

        items.push({ name, description: desc, qty, cost, price, subtotal });

        // Update Inventory Master (Palm & Coco or Dedicated) gamit ang Product + Description Key
        let foundInPalm = palmCocoMaster.find(p => p.name.toLowerCase() === name.toLowerCase() && (p.description || '').toLowerCase() === desc.toLowerCase());
        let foundInDed = dedicatedMaster.find(p => p.name.toLowerCase() === name.toLowerCase() && (p.description || '').toLowerCase() === desc.toLowerCase());

        if(foundInPalm) {
          if(location === 'Inventory Only') {
            foundInPalm.stock += qty;
            stockInHistory.unshift({ date, name, description: desc, qty, supplier: customer, uri: 'Stock In' });
          } else {
            foundInPalm.stock -= qty;
          }
          foundInPalm.cost = cost > 0 ? cost : foundInPalm.cost;
          foundInPalm.price = price > 0 ? price : foundInPalm.price;
        } else if(foundInDed) {
          if(location === 'Inventory Only') {
            foundInDed.stock += qty;
            stockInHistory.unshift({ date, name, description: desc, qty, supplier: customer, uri: 'Stock In' });
          } else {
            foundInDed.stock -= qty;
          }
          foundInDed.cost = cost > 0 ? cost : foundInDed.cost;
          foundInDed.price = price > 0 ? price : foundInDed.price;
        } else {
          // Kung wala pa sa master, idagdag sa Dedicated o Palm & Coco batay sa pangalan
          const newProd = { name, description: desc, cost, price, stock: location === 'Inventory Only' ? qty : -qty };
          dedicatedMaster.push(newProd);
          if(location === 'Inventory Only') {
            stockInHistory.unshift({ date, name, description: desc, qty, supplier: customer, uri: 'Stock In' });
          }
        }
      });

      const totalAmount = isInv ? 0 : (parseFloat(document.getElementById('totalAmount').value) || 0);
      const paymentType = isInv ? 'FULL' : document.getElementById('paymentType').value;
      const paymentMethod = isInv ? 'Cash' : document.getElementById('paymentMethod').value;
      const amountPaid = isInv ? 0 : (parseFloat(document.getElementById('amountPaidNow').value) || 0);
      const remainingBalance = isInv ? 0 : (parseFloat(document.getElementById('remainingBalance').value) || 0);
      const dueDate = document.getElementById('dueDate').value;

      let containerInfo = 'NONE';
      if(!isInv) {
        const cStatus = document.getElementById('containerStatus').value;
        if(cStatus !== 'NONE') {
          const cQty = document.getElementById('containerQty').value;
          const cRate = document.getElementById('containerDepositRate').value || 0;
          containerInfo = `${cStatus} (${cQty} pcs @ ₱${cRate})`;
        }
      }

      let multiBreakdown = { cash: 0, gcash: 0, bt: 0, other: 0 };
      if(paymentMethod === 'Multi-Payment') {
        multiBreakdown.cash = parseFloat(document.getElementById('multiCashAmt').value) || 0;
        multiBreakdown.gcash = parseFloat(document.getElementById('multiGcashAmt').value) || 0;
        multiBreakdown.bt = parseFloat(document.getElementById('multiBTAmt').value) || 0;
        multiBreakdown.other = parseFloat(document.getElementById('multiOtherAmt').value) || 0;
      }

      const netProfit = totalAmount - totalCost;

      const tx = {
        id: 'TX-' + Date.now(),
        date,
        customer,
        location,
        items,
        containerInfo,
        totalCost,
        totalAmount,
        amountPaid,
        remainingBalance,
        paymentType,
        paymentMethod,
        dueDate,
        netProfit,
        multiBreakdown,
        status: remainingBalance > 0 ? 'UNPAID' : 'PAID'
      };

      transactions.unshift(tx);
      manualSaveData();
      alert('Matagumpay na nai-save ang transaksyon at na-update ang inventory!');
      document.getElementById('posForm').reset();
      document.getElementById('saleDate').value = new Date().toISOString().split('T')[0];
      document.getElementById('posItemsBody.innerHTML').innerHTML = '';
      addPosRow();
      toggleInventoryOnlyMode();
    });

    // Render Inventory Tables with Product + Description Separation
    function renderInventoryTables() {
      const palmBody = document.getElementById('palmCocoInventoryTableBody');
      const dedBody = document.getElementById('dedicatedInventoryTableBody');
      palmBody.innerHTML = '';
      dedBody.innerHTML = '';

      let totalStockValuation = 0;

      palmCocoMaster.forEach((p, idx) => {
        // Kuwentahin ang total na nabenta para sa item na ito batay sa Product + Description
        let soldQty = 0;
        transactions.forEach(tx => {
          if(tx.location !== 'Inventory Only') {
            tx.items.forEach(it => {
              if(it.name.toLowerCase() === p.name.toLowerCase() && (it.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
                soldQty += it.qty;
              }
            });
          }
        });
        const beginning = p.stock + soldQty;
        const ending = p.stock;
        totalStockValuation += (ending * p.cost);

        palmBody.innerHTML += `
          <tr>
            <td class="fw-semibold">${p.name}</td>
            <td><span class="badge bg-light text-dark border">${p.description || 'Standard'}</span></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${p.cost}" onchange="updateMasterItem('palm', ${idx}, 'cost', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${p.price}" onchange="updateMasterItem('palm', ${idx}, 'price', this.value)"></td>
            <td class="text-center bg-light">${beginning}</td>
            <td class="text-center text-success fw-bold">+0</td>
            <td class="text-center text-danger">${soldQty}</td>
            <td class="text-center fw-bold bg-success-subtle">${ending}</td>
            <td class="col-action no-print"><button class="btn btn-sm btn-outline-danger" onclick="deleteMasterItem('palm', ${idx})"><i class="fa-solid fa-trash"></i></button></td>
          </tr>
        `;
      });

      dedicatedMaster.forEach((p, idx) => {
        let soldQty = 0;
        transactions.forEach(tx => {
          if(tx.location !== 'Inventory Only') {
            tx.items.forEach(it => {
              if(it.name.toLowerCase() === p.name.toLowerCase() && (it.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
                soldQty += it.qty;
              }
            });
          }
        });
        const beginning = p.stock + soldQty;
        const ending = p.stock;
        totalStockValuation += (ending * p.cost);

        dedBody.innerHTML += `
          <tr>
            <td class="fw-semibold">${p.name}</td>
            <td><span class="badge bg-light text-dark border">${p.description || 'Standard'}</span></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${p.cost}" onchange="updateMasterItem('ded', ${idx}, 'cost', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${p.price}" onchange="updateMasterItem('ded', ${idx}, 'price', this.value)"></td>
            <td class="text-center bg-light">${beginning}</td>
            <td class="text-center text-success fw-bold">+0</td>
            <td class="text-center text-danger">${soldQty}</td>
            <td class="text-center fw-bold bg-success-subtle">${ending}</td>
            <td class="col-action no-print"><button class="btn btn-sm btn-outline-danger" onclick="deleteMasterItem('ded', ${idx})"><i class="fa-solid fa-trash"></i></button></td>
          </tr>
        `;
      });

      document.getElementById('inventoryTableFooter').innerHTML = `
        <tr>
          <td colspan="7" class="text-end">KABUUANG HALAGA NG INVENTORY (Stock Valuation):</td>
          <td colspan="2" class="text-success fs-5">₱${totalStockValuation.toLocaleString('en-US', {minimumFractionDigits: 2})}</td>
        </tr>
      `;
    }

    function updateMasterItem(cat, idx, field, val) {
      const v = parseFloat(val) || 0;
      if(cat === 'palm') palmCocoMaster[idx][field] = v;
      if(cat === 'ded') dedicatedMaster[idx][field] = v;
      manualSaveData();
    }

    function deleteMasterItem(cat, idx) {
      if(confirm('Sigurado ka bang gusto mong tanggalin ang produktong ito sa master inventory?')) {
        if(cat === 'palm') palmCocoMaster.splice(idx, 1);
        if(cat === 'ded') dedicatedMaster.splice(idx, 1);
        manualSaveData();
        renderInventoryTables();
      }
    }

    // Add Product Modal Form Handler
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const cat = document.getElementById('newProdCategory').value;
      const name = document.getElementById('newProdName').value.trim();
      const description = document.getElementById('newProdDesc').value.trim();
      const cost = parseFloat(document.getElementById('newProdCost').value) || 0;
      const price = parseFloat(document.getElementById('newProdPrice').value) || 0;
      const stock = parseFloat(document.getElementById('newProdStock').value) || 0;

      const newProd = { name, description, cost, price, stock };
      if(cat === 'palm') {
        palmCocoMaster.push(newProd);
      } else {
        dedicatedMaster.push(newProd);
      }
      manualSaveData();
      alert('Matagumpay na naidagdag ang bagong produkto!');
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      this.reset();
      renderInventoryTables();
    });

    // Populate Return Product Dropdown
    function populateReturnProducts() {
      const cat = document.getElementById('returnCategory').value;
      const select = document.getElementById('returnProductSelect');
      select.innerHTML = '';
      const list = cat === 'palm' ? palmCocoMaster : dedicatedMaster;
      list.forEach(p => {
        select.innerHTML += `<option value="${p.name}___${p.description}">${p.name} - ${p.description || 'Standard'}</option>`;
      });
    }

    // Item Return Form Handler
    document.getElementById('returnForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('returnDate').value;
      const customer = document.getElementById('returnCustomer').value.trim();
      const cat = document.getElementById('returnCategory').value;
      const val = document.getElementById('returnProductSelect').value;
      const qty = parseFloat(document.getElementById('returnQty').value) || 0;
      const notes = document.getElementById('returnNotes').value.trim();

      const [pName, pDesc] = val.split('___');

      // Dagdagan ulit ang stock sa inventory
      const list = cat === 'palm' ? palmCocoMaster : dedicatedMaster;
      const prod = list.find(p => p.name.toLowerCase() === pName.toLowerCase() && (p.description || '').toLowerCase() === pDesc.toLowerCase());
      if(prod) {
        prod.stock += qty;
      }

      itemReturns.unshift({ date, customer, productName: pName, description: pDesc, qty, notes });
      manualSaveData();
      alert('Matagumpay na naitala ang pagka-isauli (Return) ng item!');
      bootstrap.Modal.getInstance(document.getElementById('returnModal')).hide();
      this.reset();
      renderInventoryTables();
    });

    // Daily Report Generation
    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';

      let hiwaySales = 0, hiwayProfit = 0;
      let byaheSales = 0, byaheProfit = 0;
      let totalSales = 0, txCount = 0;

      const filtered = transactions.filter(tx => tx.date === selectedDate);

      filtered.forEach((tx, index) => {
        totalSales += tx.totalAmount;
        txCount++;

        let prodNamesStr = tx.items.map(i => `${i.name} (${i.description || 'Std'}) x${i.qty}`).join(', ');

        if(tx.location === 'Hiway') {
          hiwaySales += tx.totalAmount;
          hiwayProfit += tx.netProfit;
        } else if(tx.location === 'Byahe') {
          byaheSales += tx.totalAmount;
          byaheProfit += tx.netProfit;
        }

        tbody.innerHTML += `
          <tr>
            <td>${index + 1}</td>
            <td class="fw-semibold">${tx.customer}</td>
            <td><span class="badge ${tx.location === 'Hiway' ? 'bg-primary' : tx.location === 'Byahe' ? 'bg-teal text-white' : 'bg-secondary'}">${tx.location}</span></td>
            <td>${prodNamesStr}</td>
            <td><small class="text-muted">${tx.containerInfo}</small></td>
            <td class="text-end">₱${tx.totalCost.toFixed(2)}</td>
            <td class="text-end fw-bold text-primary">₱${tx.totalAmount.toFixed(2)}</td>
            <td class="text-end text-success">₱${tx.amountPaid.toFixed(2)}</td>
            <td class="text-end text-danger">₱${tx.remainingBalance.toFixed(2)}</td>
            <td class="text-end fw-bold text-success">₱${tx.netProfit.toFixed(2)}</td>
            <td>${tx.paymentMethod}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary mb-1" onclick="openEditTxModal('${tx.id}')" title="Edit"><i class="fa-solid fa-pen"></i></button>
              <button class="btn btn-sm btn-outline-danger mb-1" onclick="deleteTransaction('${tx.id}')" title="Delete"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      document.getElementById('dailyHiwaySales').innerText = '₱' + hiwaySales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyHiwayProfit').innerText = '₱' + hiwayProfit.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyByaheSales').innerText = '₱' + byaheSales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyByaheProfit').innerText = '₱' + byaheProfit.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyTotalSales').innerText = '₱' + totalSales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyTxCount').innerText = txCount;

      // Expenses para sa araw na ito
      let dayExpSum = 0;
      standaloneExpenses.forEach(ex => {
        if(ex.date === selectedDate) {
          dayExpSum += (ex.salaryAmt || 0) + (ex.expenseAmt || 0);
        }
      });

      const subtotalNet = hiwayProfit + byaheProfit;
      const finalDayNet = subtotalNet - dayExpSum;

      document.getElementById('dailyTotalNetProfit').innerText = '₱' + finalDayNet.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('dailyExpensesDeducted').innerText = 'Exp/Sahod: ₱' + dayExpSum.toLocaleString('en-US', {minimumFractionDigits: 2});

      calculateMoneyBreakdown();
    }

    // Denomination Money Breakdown & Audit Calculation
    function calculateMoneyBreakdown() {
      let totalPcs = 0;
      let totalCashAmt = 0;

      document.querySelectorAll('.denom-count').forEach(inp => {
        const denom = parseFloat(inp.getAttribute('data-denom')) || 0;
        const count = parseFloat(inp.value) || 0;
        const sub = denom * count;
        inp.closest('tr').querySelector('.denom-subtotal').value = sub.toFixed(2);
        totalPcs += count;
        totalCashAmt += sub;
      });

      const coins = parseFloat(document.querySelector('.denom-coins').value) || 0;
      totalCashAmt += coins;

      document.getElementById('breakdownTotalPcs').innerText = totalPcs + ' pcs';
      document.getElementById('breakdownTotalAmount').innerText = '₱' + totalCashAmt.toLocaleString('en-US', {minimumFractionDigits: 2});

      const selectedDate = document.getElementById('dailyReportDate').value;
      let totalCollection = 0;
      let byaheCash = 0, gcash = 0, bt = 0, cheque = 0;
      let remainingBalSum = 0;

      transactions.forEach(tx => {
        if(tx.date === selectedDate) {
          totalCollection += tx.totalAmount;
          remainingBalSum += tx.remainingBalance;
          if(tx.paymentMethod === 'Byahe Cash') byaheCash += tx.totalAmount;
          if(tx.paymentMethod === 'GCash') gcash += tx.totalAmount;
          if(tx.paymentMethod === 'Bank Transfer') bt += tx.totalAmount;
          if(tx.paymentMethod === 'Cheque') cheque += tx.totalAmount;
          if(tx.paymentMethod === 'Multi-Payment' && tx.multiBreakdown) {
            byaheCash += (tx.multiBreakdown.other || 0);
            gcash += (tx.multiBreakdown.gcash || 0);
            bt += (tx.multiBreakdown.bt || 0);
          }
        }
      });

      // Standalone payment collections sa araw na ito
      let debtCollection = 0;
      standalonePayments.forEach(sp => {
        if(sp.date === selectedDate) {
          debtCollection += sp.amount;
          if(sp.method === 'Byahe Cash') byaheCash += sp.amount;
          else if(sp.method === 'GCash') gcash += sp.amount;
          else if(sp.method === 'Bank Transfer') bt += sp.amount;
          else if(sp.method === 'Cheque') cheque += sp.amount;
        }
      });

      let dayExp = 0;
      standaloneExpenses.forEach(ex => {
        if(ex.date === selectedDate) dayExp += (ex.salaryAmt || 0) + (ex.expenseAmt || 0);
      });

      document.getElementById('totalCollectionAll').innerText = '₱' + totalCollection.toFixed(2);
      document.getElementById('breakdownDebtPayment').innerText = '+₱' + debtCollection.toFixed(2);
      document.getElementById('lessByaheCash').innerText = '-₱' + byaheCash.toFixed(2);
      document.getElementById('lessGCash').innerText = '-₱' + gcash.toFixed(2);
      document.getElementById('lessBT').innerText = '-₱' + bt.toFixed(2);
      document.getElementById('lessCheque').innerText = '-₱' + cheque.toFixed(2);
      document.getElementById('lessExpenses').innerText = '-₱' + dayExp.toFixed(2);
      document.getElementById('lessRemainingBalance').innerText = '-₱' + remainingBalSum.toFixed(2);

      // Target Cash in Drawer = (Cash sales + Debt Collection) - non-cash methods - expenses
      let cashSales = totalCollection - (byaheCash + gcash + bt + cheque + remainingBalSum);
      let targetCashSales = cashSales + debtCollection - dayExp;
      document.getElementById('breakdownTargetSales').innerText = '₱' + targetCashSales.toFixed(2);

      const fund = parseFloat(document.getElementById('cashFundInput').value) || 0;
      const targetWithFund = targetCashSales + fund;
      document.getElementById('breakdownTargetWithFund').innerText = '₱' + targetWithFund.toFixed(2);

      const totalCountedMinusFund = totalCashAmt - fund;
      document.getElementById('totalCountedCash').innerText = '₱' + totalCountedMinusFund.toFixed(2);

      const discrepancy = totalCountedMinusFund - targetCashSales;
      const discEl = document.getElementById('cashDiscrepancy');
      const statusAlert = document.getElementById('cashStatusAlert');

      discEl.innerText = '₱' + discrepancy.toFixed(2);

      if(Math.abs(discrepancy) < 1) {
        discEl.className = 'fs-5 fw-bold text-success';
        statusAlert.className = 'alert alert-success text-center p-2 fw-bold mb-0';
        statusAlert.innerHTML = '<i class="fa-solid fa-circle-check me-1"></i> BALANSE ANG PERA SA DRAWER (No Discrepancy)';
      } else if(discrepancy > 0) {
        discEl.className = 'fs-5 fw-bold text-primary';
        statusAlert.className = 'alert alert-primary text-center p-2 fw-bold mb-0';
        statusAlert.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> OVER ANG PERA NG ₱${discrepancy.toFixed(2)}`;
      } else {
        discEl.className = 'fs-5 fw-bold text-danger';
        statusAlert.className = 'alert alert-danger text-center p-2 fw-bold mb-0';
        statusAlert.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> SHORT ANG PERA NG ₱${Math.abs(discrepancy).toFixed(2)}`;
      }
    }

    function clearMoneyBreakdown() {
      document.querySelectorAll('.denom-count').forEach(i => i.value = '');
      document.querySelectorAll('.denom-subtotal').forEach(i => i.value = '0.00');
      document.querySelector('.denom-coins').value = '';
      document.getElementById('cashFundInput').value = '';
      calculateMoneyBreakdown();
    }

    function handleEnterNext(event, input) {
      if(event.key === 'Enter') {
        event.preventDefault();
        const inputs = Array.from(document.querySelectorAll('.denom-count, .denom-coins'));
        const idx = inputs.indexOf(input);
        if(idx > -1 && idx < inputs.length - 1) {
          inputs[idx + 1].focus();
        }
      }
    }

    // Per-Day Inventory Sheet Rendering
    function renderDailyInventorySheet() {
      const selectedDate = document.getElementById('inventorySheetDate').value;
      const palmBody = document.getElementById('dailyPalmCocoSheetBody');
      const dedBody = document.getElementById('dailyDedicatedSheetBody');
      palmBody.innerHTML = '';
      dedBody.innerHTML = '';

      palmCocoMaster.forEach(p => {
        let sold = 0;
        let stockIn = 0;
        let returned = 0;

        transactions.forEach(tx => {
          if(tx.date === selectedDate) {
            tx.items.forEach(it => {
              if(it.name.toLowerCase() === p.name.toLowerCase() && (it.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
                if(tx.location === 'Inventory Only') stockIn += it.qty;
                else sold += it.qty;
              }
            });
          }
        });

        itemReturns.forEach(rt => {
          if(rt.date === selectedDate && rt.productName.toLowerCase() === p.name.toLowerCase() && (rt.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
            returned += rt.qty;
          }
        });

        stockInHistory.forEach(si => {
          if(si.date === selectedDate && si.name.toLowerCase() === p.name.toLowerCase() && (si.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
            stockIn += si.qty;
          }
        });

        const ending = p.stock;
        const beginning = ending - stockIn + sold - returned;

        palmBody.innerHTML += `
          <tr>
            <td class="fw-semibold">${p.name}</td>
            <td><span class="badge bg-light text-dark border">${p.description || 'Standard'}</span></td>
            <td class="text-center bg-light">${beginning}</td>
            <td class="text-center text-success fw-bold">+${stockIn}</td>
            <td class="text-center text-info">${returned}</td>
            <td class="text-center text-danger">${sold}</td>
            <td class="text-center fw-bold table-success">${ending}</td>
          </tr>
        `;
      });

      dedicatedMaster.forEach(p => {
        let sold = 0;
        let stockIn = 0;
        let returned = 0;

        transactions.forEach(tx => {
          if(tx.date === selectedDate) {
            tx.items.forEach(it => {
              if(it.name.toLowerCase() === p.name.toLowerCase() && (it.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
                if(tx.location === 'Inventory Only') stockIn += it.qty;
                else sold += it.qty;
              }
            });
          }
        });

        itemReturns.forEach(rt => {
          if(rt.date === selectedDate && rt.productName.toLowerCase() === p.name.toLowerCase() && (rt.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
            returned += rt.qty;
          }
        });

        stockInHistory.forEach(si => {
          if(si.date === selectedDate && si.name.toLowerCase() === p.name.toLowerCase() && (si.description || '').toLowerCase() === (p.description || '').toLowerCase()) {
            stockIn += si.qty;
          }
        });

        const ending = p.stock;
        const beginning = ending - stockIn + sold - returned;

        dedBody.innerHTML += `
          <tr>
            <td class="fw-semibold">${p.name}</td>
            <td><span class="badge bg-light text-dark border">${p.description || 'Standard'}</span></td>
            <td class="text-center bg-light">${beginning}</td>
            <td class="text-center text-success fw-bold">+${stockIn}</td>
            <td class="text-center text-info">${returned}</td>
            <td class="text-center text-danger">${sold}</td>
            <td class="text-center fw-bold table-success">${ending}</td>
          </tr>
        `;
      });
    }

    // Stock-In History Render
    function renderStockInHistory() {
      const q = document.getElementById('searchStockInInput').value.toLowerCase();
      const tbody = document.getElementById('stockInHistoryBody');
      tbody.innerHTML = '';

      const filtered = stockInHistory.filter(s => 
        s.name.toLowerCase().includes(q) || (s.description || '').toLowerCase().includes(q) || s.supplier.toLowerCase().includes(q) || s.date.includes(q)
      );

      filtered.forEach(s => {
        tbody.innerHTML += `
          <tr>
            <td>${s.date}</td>
            <td class="fw-semibold">${s.name}</td>
            <td><span class="badge bg-light text-dark border">${s.description || 'Standard'}</span></td>
            <td class="text-center text-success fw-bold">+${s.qty}</td>
            <td>${s.supplier}</td>
            <td><span class="badge bg-secondary">${s.uri}</span></td>
          </tr>
        `;
      });
    }

    // Customer Sales Log Render
    function renderCustomerSalesLog() {
      const tbody = document.getElementById('customerSalesLogBody');
      tbody.innerHTML = '';

      transactions.forEach(tx => {
        if(tx.location !== 'Inventory Only') {
          tx.items.forEach(it => {
            tbody.innerHTML += `
              <tr>
                <td>${tx.date}</td>
                <td class="fw-semibold">${tx.customer}</td>
                <td><span class="badge ${tx.location === 'Hiway' ? 'bg-primary' : 'bg-teal text-white'}">${tx.location}</span></td>
                <td>${it.name}</td>
                <td><span class="badge bg-light text-dark border">${it.description || 'Standard'}</span></td>
                <td class="text-center">${it.qty}</td>
                <td class="text-end">₱${it.cost.toFixed(2)}</td>
                <td class="text-end">₱${it.price.toFixed(2)}</td>
                <td class="text-end fw-bold">₱${it.subtotal.toFixed(2)}</td>
              </tr>
            `;
          });
        }
      });
    }

    // Utang & Payments Tab
    function renderCreditTable() {
      const creditBody = document.getElementById('creditTableBody');
      const paidBody = document.getElementById('paidHistoryTableBody');
      creditBody.innerHTML = '';
      paidBody.innerHTML = '';

      transactions.forEach(tx => {
        if(tx.location !== 'Inventory Only') {
          let prodStr = tx.items.map(i => `${i.name} (${i.description || 'Std'}) x${i.qty}`).join(', ');

          if(tx.remainingBalance > 0) {
            creditBody.innerHTML += `
              <tr>
                <td class="fw-bold">${tx.customer}</td>
                <td><span class="badge bg-secondary">${tx.location}</span></td>
                <td>${prodStr}</td>
                <td class="text-end">₱${tx.totalCost.toFixed(2)}</td>
                <td class="text-end text-success">₱${tx.amountPaid.toFixed(2)}</td>
                <td class="text-end text-danger fw-bold">₱${tx.remainingBalance.toFixed(2)}</td>
                <td>${tx.dueDate || '-'}</td>
                <td><span class="badge bg-danger">UNPAID</span></td>
                <td class="no-print">
                  <button class="btn btn-sm btn-success fw-bold" onclick="openPaymentModal('${tx.id}')"><i class="fa-solid fa-peso-sign me-1"></i>Magbayad</button>
                </td>
              </tr>
            `;
          } else {
            paidBody.innerHTML += `
              <tr>
                <td class="fw-semibold">${tx.customer}</td>
                <td>${prodStr}</td>
                <td class="text-end">₱${tx.totalAmount.toFixed(2)}</td>
                <td class="text-end text-success">₱${tx.totalAmount.toFixed(2)}</td>
                <td><span class="badge bg-success">PAID FULL</span></td>
                <td class="text-center no-print">
                  <button class="btn btn-sm btn-outline-primary" onclick="openEditTxModal('${tx.id}')"><i class="fa-solid fa-pen"></i></button>
                </td>
              </tr>
            `;
          }
        }
      });
    }

    function openPaymentModal(txId) {
      const tx = transactions.find(t => t.id === txId);
      if(tx) {
        document.getElementById('payTxId').value = tx.id;
        document.getElementById('payCustomerName').value = tx.customer;
        document.getElementById('payTotalAmount').value = tx.totalAmount.toFixed(2);
        document.getElementById('payRemainingBalance').value = tx.remainingBalance.toFixed(2);
        document.getElementById('payAmountNow').value = tx.remainingBalance.toFixed(2);
        new bootstrap.Modal(document.getElementById('paymentModal')).show();
      }
    }

    document.getElementById('paymentForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const txId = document.getElementById('payTxId').value;
      const amtNow = parseFloat(document.getElementById('payAmountNow').value) || 0;
      const payDate = document.getElementById('payDate').value;
      const payMethod = document.getElementById('payMethod').value;

      const tx = transactions.find(t => t.id === txId);
      if(tx) {
        tx.amountPaid += amtNow;
        tx.remainingBalance = Math.max(0, tx.totalAmount - tx.amountPaid);
        if(tx.remainingBalance === 0) tx.status = 'PAID';

        // Magdagdag din sa standalone payment history para sa araw na ito
        standalonePayments.unshift({
          date: payDate,
          customer: tx.customer,
          amount: amtNow,
          method: payMethod,
          notes: `Bayad sa utang para sa ${tx.id}`
        });

        manualSaveData();
        alert('Matagumpay na naitala ang bayad!');
        bootstrap.Modal.getInstance(document.getElementById('paymentModal')).hide();
        renderCreditTable();
        renderStandalonePayments();
      }
    });

    function renderStandalonePayments() {
      const q = document.getElementById('searchStandalonePaymentInput') ? document.getElementById('searchStandalonePaymentInput').value.toLowerCase() : '';
      const tbody = document.getElementById('standalonePaymentTableBody');
      const tfoot = document.getElementById('standalonePaymentTableFooter');
      tbody.innerHTML = '';

      let subTotal = 0;
      const filtered = standalonePayments.filter(sp => 
        sp.customer.toLowerCase().includes(q) || sp.date.includes(q) || (sp.notes || '').toLowerCase().includes(q)
      );

      filtered.forEach((sp, idx) => {
        subTotal += sp.amount;
        tbody.innerHTML += `
          <tr>
            <td>${sp.date}</td>
            <td class="fw-semibold">${sp.customer}</td>
            <td><span class="badge bg-secondary">${sp.method}</span></td>
            <td class="text-end fw-bold text-success">₱${sp.amount.toFixed(2)}</td>
            <td><small class="text-muted">${sp.notes || '-'}</small></td>
            <td class="col-action no-print text-center">
              <button class="btn btn-sm btn-outline-danger" onclick="deleteStandalonePayment(${idx})"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      tfoot.innerHTML = `
        <tr>
          <td colspan="3" class="text-end">KABUUANG KOLEKSIYON SA MGA LUMANG UTANG:</td>
          <td class="text-end text-success fs-5">₱${subTotal.toLocaleString('en-US', {minimumFractionDigits: 2})}</td>
          <td colspan="2"></td>
        </tr>
      `;
    }

    document.getElementById('standalonePaymentForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('stdPayDate').value;
      const customer = document.getElementById('stdPayCustomer').value.trim();
      const amount = parseFloat(document.getElementById('stdPayAmount').value) || 0;
      const method = document.getElementById('stdPayMethod').value;
      const notes = document.getElementById('stdPayNotes').value.trim();

      standalonePayments.unshift({ date, customer, amount, method, notes });
      manualSaveData();
      alert('Matagumpay na naidagdag ang manual payment collection!');
      bootstrap.Modal.getInstance(document.getElementById('standalonePaymentModal')).hide();
      this.reset();
      document.getElementById('stdPayDate').value = new Date().toISOString().split('T')[0];
      renderStandalonePayments();
    });

    function deleteStandalonePayment(idx) {
      if(confirm('Sigurado ka bang gusto mong burahin ang record na ito?')) {
        standalonePayments.splice(idx, 1);
        manualSaveData();
        renderStandalonePayments();
      }
    }

    // Customer Order Lookup & Last Price
    function searchCustomerOrder() {
      const q = document.getElementById('searchCustomerInput').value.trim().toLowerCase();
      const container = document.getElementById('searchResultContainer');
      const noFound = document.getElementById('noCustomerFound');

      const matches = transactions.filter(tx => tx.customer.toLowerCase().includes(q));

      if(matches.length > 0) {
        container.style.display = 'block';
        noFound.classList.add('d-none');

        // Huling transaksyon ang nasa unahan
        const last = matches[0];
        document.getElementById('lastOrderCustomer').innerText = last.customer;
        document.getElementById('lastOrderDate').innerText = last.date;
        document.getElementById('lastOrderContainer').innerText = last.containerInfo;
        document.getElementById('lastOrderTotal').innerText = '₱' + last.totalAmount.toFixed(2);
        document.getElementById('lastOrderPaid').innerText = '₱' + last.amountPaid.toFixed(2);
        document.getElementById('lastOrderBalance').innerText = '₱' + last.remainingBalance.toFixed(2);

        const badge = document.getElementById('lastOrderBadge');
        badge.innerText = last.status;
        badge.className = `badge ${last.status === 'PAID' ? 'bg-success' : 'bg-danger'} fs-6`;

        const itemsBody = document.getElementById('lastOrderItemsBody');
        itemsBody.innerHTML = '';
        last.items.forEach(it => {
          itemsBody.innerHTML += `
            <tr>
              <td class="fw-semibold">${it.name}</td>
              <td><span class="badge bg-light text-dark border">${it.description || 'Standard'}</span></td>
              <td class="text-center">${it.qty}</td>
              <td class="text-end">₱${it.price.toFixed(2)}</td>
              <td class="text-end fw-bold">₱${it.subtotal.toFixed(2)}</td>
            </tr>
          `;
        });

        const historyBody = document.getElementById('customerHistoryBody');
        historyBody.innerHTML = '';
        matches.forEach(tx => {
          let prodStr = tx.items.map(i => `${i.name} (${i.description || 'Std'}) x${i.qty} @ ₱${i.price}`).join(', ');
          historyBody.innerHTML += `
            <tr>
              <td>${tx.date}</td>
              <td><span class="badge bg-secondary">${tx.location}</span></td>
              <td>${prodStr}</td>
              <td><small>${tx.containerInfo}</small></td>
              <td class="text-end fw-bold">₱${tx.totalAmount.toFixed(2)}</td>
              <td class="text-end text-success">₱${tx.amountPaid.toFixed(2)}</td>
              <td class="text-end text-danger">₱${tx.remainingBalance.toFixed(2)}</td>
              <td class="text-end text-success">₱${tx.netProfit.toFixed(2)}</td>
              <td><span class="badge ${tx.status === 'PAID' ? 'bg-success' : 'bg-danger'}">${tx.status}</span></td>
              <td class="text-center no-print">
                <button class="btn btn-sm btn-outline-primary" onclick="openEditTxModal('${tx.id}')"><i class="fa-solid fa-pen"></i></button>
              </td>
            </tr>
          `;
        });
      } else {
        container.style.display = 'none';
        noFound.classList.remove('d-none');
      }
    }

    // Salary & Expenses Standalone Ledger Tab
    function toggleExpenseViewMode() {
      const mode = document.getElementById('expenseViewMode').value;
      document.getElementById('standaloneExpenseMonth').style.display = mode === 'month' ? '' : 'none';
      document.getElementById('standaloneExpenseDate').style.display = mode === 'day' ? '' : 'none';
      renderStandaloneExpensesLedger();
    }

    function addStandaloneExpenseRow() {
      const date = document.getElementById('standaloneExpenseDate').value || new Date().toISOString().split('T')[0];
      standaloneExpenses.unshift({ date, salaryName: '', salaryAmt: 0, expenseName: '', expenseAmt: 0 });
      manualSaveData();
      renderStandaloneExpensesLedger();
    }

    function renderStandaloneExpensesLedger() {
      const mode = document.getElementById('expenseViewMode').value;
      const mVal = document.getElementById('standaloneExpenseMonth').value;
      const dVal = document.getElementById('standaloneExpenseDate').value;
      const q = document.getElementById('searchStandaloneExpenseInput').value.toLowerCase();
      const tbody = document.getElementById('standaloneExpenseTableBody');
      const tfoot = document.getElementById('standaloneExpenseTableFooter');
      tbody.innerHTML = '';

      let totalSal = 0, totalExp = 0;

      const filtered = standaloneExpenses.filter((ex, idx) => {
        let matchFilter = true;
        if(mode === 'month' && mVal) matchFilter = ex.date.startsWith(mVal);
        if(mode === 'day' && dVal) matchFilter = ex.date === dVal;
        const matchQ = (ex.salaryName || '').toLowerCase().includes(q) || (ex.expenseName || '').toLowerCase().includes(q) || ex.date.includes(q);
        return matchFilter && matchQ;
      });

      filtered.forEach((ex, idx) => {
        // Hanapin ang tunay na index sa buong standaloneExpenses array
        const realIdx = standaloneExpenses.indexOf(ex);
        totalSal += (ex.salaryAmt || 0);
        totalExp += (ex.expenseAmt || 0);

        tbody.innerHTML += `
          <tr>
            <td><input type="date" class="form-control form-control-sm" value="${ex.date}" onchange="updateExpenseField(${realIdx}, 'date', this.value)"></td>
            <td><input type="text" class="form-control form-control-sm" placeholder="Pangalan ng Sahod" value="${ex.salaryName || ''}" onchange="updateExpenseField(${realIdx}, 'salaryName', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${ex.salaryAmt || 0}" onchange="updateExpenseField(${realIdx}, 'salaryAmt', this.value)"></td>
            <td><input type="text" class="form-control form-control-sm" placeholder="Pangalan ng Gastos" value="${ex.expenseName || ''}" onchange="updateExpenseField(${realIdx}, 'expenseName', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-end" value="${ex.expenseAmt || 0}" onchange="updateExpenseField(${realIdx}, 'expenseAmt', this.value)"></td>
            <td class="col-action no-print text-center"><button class="btn btn-sm btn-outline-danger" onclick="deleteStandaloneExpense(${realIdx})"><i class="fa-solid fa-trash"></i></button></td>
          </tr>
        `;
      });

      document.getElementById('totalSalarySumDisplay').innerText = '₱' + totalSal.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('totalExpenseSumDisplay').innerText = '₱' + totalExp.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('totalCombinedExpenseDisplay').innerText = '₱' + (totalSal + totalExp).toLocaleString('en-US', {minimumFractionDigits: 2});

      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">SUBTOTAL:</td>
          <td class="text-end text-danger">₱${totalSal.toFixed(2)}</td>
          <td class="text-end">SUBTOTAL:</td>
          <td class="text-end text-warning-emphasis">₱${totalExp.toFixed(2)}</td>
          <td></td>
        </tr>
      `;
    }

    function updateExpenseField(idx, field, val) {
      if(field === 'salaryAmt' || field === 'expenseAmt') {
        standaloneExpenses[idx][field] = parseFloat(val) || 0;
      } else {
        standaloneExpenses[idx][field] = val;
      }
      manualSaveData();
      renderStandaloneExpensesLedger();
    }

    function deleteStandaloneExpense(idx) {
      if(confirm('Sigurado ka bang gusto mong burahin ang line item na ito?')) {
        standaloneExpenses.splice(idx, 1);
        manualSaveData();
        renderStandaloneExpensesLedger();
      }
    }

    // D/Eco Boss Standalone Ledger Tab
    function toggleBossViewMode() {
      const mode = document.getElementById('bossViewMode').value;
      document.getElementById('standaloneBossMonth').style.display = mode === 'month' ? '' : 'none';
      document.getElementById('standaloneBossDate').style.display = mode === 'day' ? '' : 'none';
      renderStandaloneBossLedger();
    }

    function renderStandaloneBossLedger() {
      const mode = document.getElementById('bossViewMode').value;
      const mVal = document.getElementById('standaloneBossMonth').value;
      const dVal = document.getElementById('standaloneBossDate').value;
      const q = document.getElementById('searchStandaloneBossInput').value.toLowerCase();
      const tbody = document.getElementById('standaloneBossTableBody');
      const tfoot = document.getElementById('standaloneBossTableFooter');
      tbody.innerHTML = '';

      let totalAdd = 0, totalSub = 0;

      const filtered = bossAdjustments.filter((b, idx) => {
        let matchFilter = true;
        if(mode === 'month' && mVal) matchFilter = b.date.startsWith(mVal);
        if(mode === 'day' && dVal) matchFilter = b.date === dVal;
        const matchQ = b.notes.toLowerCase().includes(q) || b.date.includes(q) || b.type.toLowerCase().includes(q);
        return matchFilter && matchQ;
      });

      filtered.forEach((b, idx) => {
        const realIdx = bossAdjustments.indexOf(b);
        if(b.type === 'ADD') totalAdd += b.amount;
        else totalSub += b.amount;

        tbody.innerHTML += `
          <tr>
            <td>${b.date}</td>
            <td><span class="badge ${b.type === 'ADD' ? 'bg-success' : 'bg-danger'}">${b.type === 'ADD' ? 'Capital In (+)' : 'Withdrawal (-)'}</span> ${b.notes}</td>
            <td class="text-end fw-bold ${b.type === 'ADD' ? 'text-success' : 'text-danger'}">₱${b.amount.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary" onclick="openEditBossModal(${realIdx})"><i class="fa-solid fa-pen"></i></button>
              <button class="btn btn-sm btn-outline-danger" onclick="deleteBossAdjustment(${realIdx})"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      const netBoss = totalAdd - totalSub;
      document.getElementById('totalBossAddDisplay').innerText = '+₱' + totalAdd.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('totalBossSubDisplay').innerText = '-₱' + totalSub.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('totalBossNetDisplay').innerText = '₱' + netBoss.toLocaleString('en-US', {minimumFractionDigits: 2});

      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">NET BOSS BALANCE:</td>
          <td colspan="2" class="text-primary fs-5">₱${netBoss.toFixed(2)}</td>
        </tr>
      `;
    }

    // Boss Form Submission
    document.getElementById('bossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const date = document.getElementById('bossDate').value;
      const type = document.getElementById('bossType').value;
      const amount = parseFloat(document.getElementById('bossAmount').value) || 0;
      const notes = document.getElementById('bossNotes').value.trim();

      bossAdjustments.unshift({ date, type, amount, notes });
      manualSaveData();
      alert('Matagumpay na naidagdag ang D/Eco Boss adjustment!');
      bootstrap.Modal.getInstance(document.getElementById('bossModal')).hide();
      this.reset();
      document.getElementById('bossDate').value = new Date().toISOString().split('T')[0];
      renderStandaloneBossLedger();
    });

    function openEditBossModal(idx) {
      const b = bossAdjustments[idx];
      document.getElementById('editBossIndex').value = idx;
      document.getElementById('editBossDate').value = b.date;
      document.getElementById('editBossType').value = b.type;
      document.getElementById('editBossAmount').value = b.amount;
      document.getElementById('editBossNotes').value = b.notes;
      new bootstrap.Modal(document.getElementById('editBossModal')).show();
    }

    document.getElementById('editBossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const idx = parseInt(document.getElementById('editBossIndex').value);
      bossAdjustments[idx] = {
        date: document.getElementById('editBossDate').value,
        type: document.getElementById('editBossType').value,
        amount: parseFloat(document.getElementById('editBossAmount').value) || 0,
        notes: document.getElementById('editBossNotes').value.trim()
      };
      manualSaveData();
      alert('Na-update na ang Boss Adjustment!');
      bootstrap.Modal.getInstance(document.getElementById('editBossModal')).hide();
      renderStandaloneBossLedger();
    });

    function deleteBossAdjustment(idx) {
      if(confirm('Sigurado ka bang gusto mong burahin ang adjustment na ito?')) {
        bossAdjustments.splice(idx, 1);
        manualSaveData();
        renderStandaloneBossLedger();
      }
    }

    // Monthly Audit Tab
    function generateMonthlyAudit() {
      const mStr = document.getElementById('auditMonth').value; // e.g. "2026-10"
      if(!mStr) return;

      let hiwaySales = 0, hiwayCost = 0, hiwayNet = 0;
      let byaheSales = 0, byaheCost = 0, byaheNet = 0;
      let totalSales = 0, totalCost = 0;

      transactions.forEach(tx => {
        if(tx.date.startsWith(mStr) && tx.location !== 'Inventory Only') {
          totalSales += tx.totalAmount;
          totalCost += tx.totalCost;
          if(tx.location === 'Hiway') {
            hiwaySales += tx.totalAmount;
            hiwayCost += tx.totalCost;
            hiwayNet += tx.netProfit;
          } else if(tx.location === 'Byahe') {
            byaheSales += tx.totalAmount;
            byaheCost += tx.totalCost;
            byaheNet += tx.netProfit;
          }
        }
      });

      document.getElementById('auditHiwaySales').innerText = '₱' + hiwaySales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('auditHiwayCost').innerText = '₱' + hiwayCost.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('auditHiwayNetProfit').innerText = '₱' + hiwayNet.toLocaleString('en-US', {minimumFractionDigits: 2});

      document.getElementById('auditByaheSales').innerText = '₱' + byaheSales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('auditByaheCost').innerText = '₱' + byaheCost.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('auditByaheNetProfit').innerText = '₱' + byaheNet.toLocaleString('en-US', {minimumFractionDigits: 2});

      document.getElementById('auditTotalSales').innerText = '₱' + totalSales.toLocaleString('en-US', {minimumFractionDigits: 2});
      document.getElementById('auditTotalCost').innerText = '₱' + totalCost.toLocaleString('en-US', {minimumFractionDigits: 2});

      const subtotalNet = hiwayNet + byaheNet;
      document.getElementById('auditGrossProfit').innerText = '₱' + subtotalNet.toLocaleString('en-US', {minimumFractionDigits: 2});

      // Total Expenses para sa buwang ito
      let monthlyExp = 0;
      standaloneExpenses.forEach(ex => {
        if(ex.date.startsWith(mStr)) {
          monthlyExp += (ex.salaryAmt || 0) + (ex.expenseAmt || 0);
        }
      });
      document.getElementById('auditExpenses').innerText = '₱' + monthlyExp.toLocaleString('en-US', {minimumFractionDigits: 2});

      // Total Boss Adjustments para sa buwang ito
      let bossNetMonth = 0;
      bossAdjustments.forEach(b => {
        if(b.date.startsWith(mStr)) {
          if(b.type === 'ADD') bossNetMonth += b.amount;
          else bossNetMonth -= b.amount;
        }
      });

      const finalNetProfit = subtotalNet - monthlyExp + bossNetMonth;
      document.getElementById('auditNetProfit').innerText = '₱' + finalNetProfit.toLocaleString('en-US', {minimumFractionDigits: 2});

      renderDailyAuditBreakdown(mStr);
      renderExpensesTable();
      renderBossLogsTable(mStr);
    }

    function renderDailyAuditBreakdown(mStr) {
      const tbody = document.getElementById('auditDailyBreakdownBody');
      tbody.innerHTML = '';

      // Kunin ang lahat ng natatanging petsa sa buwang ito
      let datesSet = new Set();
      transactions.forEach(tx => { if(tx.date.startsWith(mStr)) datesSet.add(tx.date); });
      standaloneExpenses.forEach(ex => { if(ex.date.startsWith(mStr)) datesSet.add(ex.date); });

      const sortedDates = Array.from(datesSet).sort();

      sortedDates.forEach(dt => {
        let hNet = 0, bNet = 0;
        transactions.forEach(tx => {
          if(tx.date === dt && tx.location !== 'Inventory Only') {
            if(tx.location === 'Hiway') hNet += tx.netProfit;
            if(tx.location === 'Byahe') bNet += tx.netProfit;
          }
        });

        let dayExp = 0;
        standaloneExpenses.forEach(ex => {
          if(ex.date === dt) dayExp += (ex.salaryAmt || 0) + (ex.expenseAmt || 0);
        });

        const subNet = hNet + bNet;
        const finalNet = subNet - dayExp;

        tbody.innerHTML += `
          <tr>
            <td class="fw-semibold">${dt}</td>
            <td class="text-end text-primary">₱${hNet.toFixed(2)}</td>
            <td class="text-end text-teal" style="color: #00897b;">₱${bNet.toFixed(2)}</td>
            <td class="text-end fw-bold">₱${subNet.toFixed(2)}</td>
            <td class="text-end fw-bold text-success">₱${finalNet.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary" onclick="goToDailyReportDate('${dt}')"><i class="fa-solid fa-eye me-1"></i>View</button>
            </td>
          </tr>
        `;
      });
    }

    function goToDailyReportDate(dt) {
      document.getElementById('dailyReportDate').value = dt;
      generateDailyReport();
      const triggerEl = document.querySelector('#daily-tab');
      const tab = new bootstrap.Tab(triggerEl);
      tab.show();
    }

    function renderExpensesTable() {
      const mStr = document.getElementById('auditMonth').value;
      const q = document.getElementById('searchExpenseInput').value.toLowerCase();
      const tbody = document.getElementById('expenseTableBody');
      const tfoot = document.getElementById('expenseTableFooter');
      tbody.innerHTML = '';

      let totSal = 0, totExp = 0;

      const filtered = standaloneExpenses.filter(ex => {
        const mMatch = mStr ? ex.date.startsWith(mStr) : true;
        const qMatch = (ex.salaryName || '').toLowerCase().includes(q) || (ex.expenseName || '').toLowerCase().includes(q) || ex.date.includes(q);
        return mMatch && qMatch;
      });

      filtered.forEach(ex => {
        const realIdx = standaloneExpenses.indexOf(ex);
        totSal += (ex.salaryAmt || 0);
        totExp += (ex.expenseAmt || 0);

        tbody.innerHTML += `
          <tr>
            <td>${ex.date}</td>
            <td>${ex.salaryName || '-'}</td>
            <td class="text-end text-danger">₱${(ex.salaryAmt || 0).toFixed(2)}</td>
            <td>${ex.expenseName || '-'}</td>
            <td class="text-end text-warning-emphasis">₱${(ex.expenseAmt || 0).toFixed(2)}</td>
            <td class="col-action no-print text-center"><button class="btn btn-sm btn-outline-danger" onclick="deleteStandaloneExpense(${realIdx}); generateMonthlyAudit();"><i class="fa-solid fa-trash"></i></button></td>
          </tr>
        `;
      });

      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">KABUUANG SAHOD:</td>
          <td class="text-end text-danger">₱${totSal.toFixed(2)}</td>
          <td class="text-end">KABUUANG GASTOS:</td>
          <td class="text-end text-warning-emphasis">₱${totExp.toFixed(2)}</td>
          <td></td>
        </tr>
      `;
    }

    function addExpenseRow() {
      addStandaloneExpenseRow();
      generateMonthlyAudit();
    }

    function renderBossLogsTable(mStr) {
      const q = document.getElementById('searchBossInput') ? document.getElementById('searchBossInput').value.toLowerCase() : '';
      const tbody = document.getElementById('bossLogsBody');
      const tfoot = document.getElementById('bossLogsFooter');
      tbody.innerHTML = '';

      let totalAdd = 0, totalSub = 0;

      const filtered = bossAdjustments.filter(b => {
        const mMatch = mStr ? b.date.startsWith(mStr) : true;
        const qMatch = b.notes.toLowerCase().includes(q) || b.date.includes(q);
        return mMatch && qMatch;
      });

      filtered.forEach(b => {
        const realIdx = bossAdjustments.indexOf(b);
        if(b.type === 'ADD') totalAdd += b.amount;
        else totalSub += b.amount;

        tbody.innerHTML += `
          <tr>
            <td>${b.date}</td>
            <td><span class="badge ${b.type === 'ADD' ? 'bg-success' : 'bg-danger'}">${b.type === 'ADD' ? 'Addition (+)' : 'Withdrawal (-)'}</span> ${b.notes}</td>
            <td class="text-end fw-bold ${b.type === 'ADD' ? 'text-success' : 'text-danger'}">₱${b.amount.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary" onclick="openEditBossModal(${realIdx})"><i class="fa-solid fa-pen"></i></button>
              <button class="btn btn-sm btn-outline-danger" onclick="deleteBossAdjustment(${realIdx}); generateMonthlyAudit();"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      const net = totalAdd - totalSub;
      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">NET BOSS ADJUSTMENT SA BUWANG ITO:</td>
          <td colspan="2" class="text-primary fs-6">₱${net.toFixed(2)}</td>
        </tr>
      `;
    }

    // Global Search All Transactions
    function renderGlobalSearchResults() {
      const q = document.getElementById('globalSearchInput').value.trim().toLowerCase();
      const wrapper = document.getElementById('globalSearchResultsWrapper');
      const tbody = document.getElementById('globalSearchResultsBody');
      tbody.innerHTML = '';

      if(!q) {
        wrapper.style.display = 'none';
        return;
      }

      wrapper.style.display = 'block';
      let matches = transactions.filter(tx => 
        tx.customer.toLowerCase().includes(q) || tx.date.includes(q) || tx.id.toLowerCase().includes(q) ||
        tx.items.some(it => it.name.toLowerCase().includes(q) || (it.description || '').toLowerCase().includes(q))
      );

      matches.forEach(tx => {
        let prodStr = tx.items.map(i => `${i.name} (${i.description || 'Std'}) x${i.qty}`).join(', ');
        tbody.innerHTML += `
          <tr>
            <td>${tx.date}</td>
            <td class="fw-semibold">${tx.customer}</td>
            <td><span class="badge bg-secondary">${tx.location}</span></td>
            <td>${prodStr}</td>
            <td class="text-end fw-bold">₱${tx.totalAmount.toFixed(2)}</td>
            <td class="text-end text-success">₱${tx.amountPaid.toFixed(2)}</td>
            <td class="text-end text-danger">₱${tx.remainingBalance.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary" onclick="openEditTxModal('${tx.id}')"><i class="fa-solid fa-pen"></i> Edit</button>
            </td>
          </tr>
        `;
      });
    }

    function clearGlobalSearch() {
      document.getElementById('globalSearchInput').value = '';
      document.getElementById('globalSearchResultsWrapper').style.display = 'none';
    }

    // Edit Transaction Modal Functions
    function openEditTxModal(txId) {
      const tx = transactions.find(t => t.id === txId);
      if(tx) {
        document.getElementById('editTxId').value = tx.id;
        document.getElementById('editTxDate').value = tx.date;
        document.getElementById('editCustomerName').value = tx.customer;
        document.getElementById('editLocation').value = tx.location;
        document.getElementById('editProduct').value = tx.items.map(i => `${i.name} (${i.description || 'Std'}) x${i.qty}`).join(', ');
        document.getElementById('editContainerInfo').value = tx.containerInfo;
        document.getElementById('editTotalCost').value = tx.totalCost.toFixed(2);
        document.getElementById('editTotal').value = tx.totalAmount.toFixed(2);
        document.getElementById('editPaid').value = tx.amountPaid.toFixed(2);
        document.getElementById('editBalance').value = tx.remainingBalance.toFixed(2);
        new bootstrap.Modal(document.getElementById('editTransactionModal')).show();
      }
    }

    function calculateEditBalance() {
      const tot = parseFloat(document.getElementById('editTotal').value) || 0;
      const pd = parseFloat(document.getElementById('editPaid').value) || 0;
      document.getElementById('editBalance').value = Math.max(0, tot - pd).toFixed(2);
    }

    document.getElementById('editTransactionForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const txId = document.getElementById('editTxId').value;
      const tx = transactions.find(t => t.id === txId);
      if(tx) {
        tx.date = document.getElementById('editTxDate').value;
        tx.customer = document.getElementById('editCustomerName').value.trim();
        tx.location = document.getElementById('editLocation').value;
        tx.containerInfo = document.getElementById('editContainerInfo').value.trim();
        tx.totalCost = parseFloat(document.getElementById('editTotalCost').value) || 0;
        tx.totalAmount = parseFloat(document.getElementById('editTotal').value) || 0;
        tx.amountPaid = parseFloat(document.getElementById('editPaid').value) || 0;
        tx.remainingBalance = parseFloat(document.getElementById('editBalance').value) || 0;
        tx.netProfit = tx.totalAmount - tx.totalCost;
        tx.status = tx.remainingBalance > 0 ? 'UNPAID' : 'PAID';

        manualSaveData();
        alert('Matagumpay na na-update ang transaksyon!');
        bootstrap.Modal.getInstance(document.getElementById('editTransactionModal')).hide();
        generateDailyReport();
        renderCreditTable();
        renderInventoryTables();
      }
    });

    function deleteTransaction(txId) {
      if(confirm('Sigurado ka bang gusto mong burahin ang transaksyong ito? Awtomatikong ibabalik ang stock sa inventory.')) {
        const idx = transactions.findIndex(t => t.id === txId);
        if(idx > -1) {
          const tx = transactions[idx];
          // Ibalik ang stock sa master inventory
          if(tx.location !== 'Inventory Only') {
            tx.items.forEach(it => {
              let f1 = palmCocoMaster.find(p => p.name.toLowerCase() === it.name.toLowerCase() && (p.description || '').toLowerCase() === (it.description || '').toLowerCase());
              let f2 = dedicatedMaster.find(p => p.name.toLowerCase() === it.name.toLowerCase() && (p.description || '').toLowerCase() === (it.description || '').toLowerCase());
              if(f1) f1.stock += it.qty;
              if(f2) f2.stock += it.qty;
            });
          }
          transactions.splice(idx, 1);
          manualSaveData();
          generateDailyReport();
          renderCreditTable();
          renderInventoryTables();
          alert('Nabura na ang transaksyon.');
        }
      }
    }

    // Profile & User Management Functions
    function openChangeProfileModal() {
      if(currentUser) {
        document.getElementById('profDisplayName').value = currentUser.name;
        document.getElementById('profPassword').value = '';
        new bootstrap.Modal(document.getElementById('changeProfileModal')).show();
      }
    }

    document.getElementById('changeProfileForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const newName = document.getElementById('profDisplayName').value.trim();
      const newPass = document.getElementById('profPassword').value.trim();

      currentUser.name = newName;
      if(newPass) currentUser.password = newPass;

      // Update sa usersList
      const uIdx = usersList.findIndex(u => u.username === currentUser.username);
      if(uIdx > -1) usersList[uIdx] = currentUser;

      localStorage.setItem('rmv_users', JSON.stringify(usersList));
      localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
      alert('Matagumpay na na-update ang iyong profile!');
      bootstrap.Modal.getInstance(document.getElementById('changeProfileModal')).hide();
      checkLoginStatus();
    });

    function openUserManagementModal() {
      renderUsersList();
      new bootstrap.Modal(document.getElementById('userManagementModal')).show();
    }

    function renderUsersList() {
      const tbody = document.getElementById('userListBody');
      tbody.innerHTML = '';
      usersList.forEach((u, idx) => {
        tbody.innerHTML += `
          <tr>
            <td>${u.name}</td>
            <td>${u.username}</td>
            <td><span class="badge ${u.role === 'Admin' ? 'bg-dark' : 'bg-secondary'}">${u.role}</span></td>
            <td class="text-center">
              ${idx > 0 ? `<button class="btn btn-sm btn-outline-danger" onclick="deleteUser(${idx})"><i class="fa-solid fa-trash"></i></button>` : '<small class="text-muted">Main Admin</small>'}
            </td>
          </tr>
        `;
      });
    }

    document.getElementById('newUserForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const name = document.getElementById('newAccName').value.trim();
      const username = document.getElementById('newAccUser').value.trim();
      const password = document.getElementById('newAccPass').value.trim();
      const role = document.getElementById('newAccRole').value;

      if(usersList.some(u => u.username === username)) {
        alert('Mayroon nang ganyang username!');
        return;
      }

      usersList.push({ name, username, password, role });
      localStorage.setItem('rmv_users', JSON.stringify(usersList));
      alert('Matagumpay na naidagdag ang bagong user!');
      this.reset();
      renderUsersList();
    });

    function deleteUser(idx) {
      if(confirm('Sigurado ka bang gusto mong burahin ang user na ito?')) {
        usersList.splice(idx, 1);
        localStorage.setItem('rmv_users', JSON.stringify(usersList));
        renderUsersList();
      }
    }
  </script>
</body>
</html>
