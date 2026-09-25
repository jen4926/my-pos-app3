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
    .credit-fields, .container-fields { display: none; background-color: #f8f9fa; border-radius: 8px; padding: 15px; margin-top: 15px; border: 1px dashed #cbd5e1; }
   
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
          <button class="nav-link" id="credit-tab" data-bs-toggle="pill" data-bs-target="#credit-content" type="button" onclick="renderCreditTable()">
            <i class="fa-solid fa-hand-holding-dollar me-1"></i> Utang & Payments
          </button>
        </li>
        <li class="nav-item">
          <button class="nav-link" id="search-tab" data-bs-toggle="pill" data-bs-target="#search-content" type="button">
            <i class="fa-solid fa-magnifying-glass me-1"></i> Order Lookup
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
                  <input type="number" min="1" id="containerQty" class="form-control" value="1" placeholder="Hal. 2">
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
                <select id="paymentMethod" class="form-select">
                  <option value="Cash">Cash</option>
                  <option value="Byahe Cash">Byahe Cash</option>
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
                <span class="text-muted small fw-bold">TOTAL COLLECTION & NET</span>
                <div class="mt-1">
                  <span class="text-success fw-bold" id="dailyTotalCollected">₱0.00</span> <small class="text-muted">(Coll)</small><br>
                  <span class="text-success fw-bold" id="dailyTotalNetProfit">₱0.00</span> <small class="text-muted">(Net)</small>
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
                    <span class="text-muted small text-danger fw-bold">Less: Unpaid Credit / Utang:</span>
                    <span class="text-danger fw-bold small" id="lessCreditBalance">-₱0.00</span>
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
            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-receipt me-2"></i>Standalone Manual Payments & Collections History (Araw-arawang Bayad)</h6>
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

      <!-- ================= 3.5 CUSTOMER ORDER LOOKUP TAB ================= -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-magnifying-glass me-2"></i>Track Customer Last Order & History</h4>
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
                <i class="fa-solid fa-search me-2"></i>Search Order
              </button>
            </div>
          </div>

          <div id="searchResultContainer" style="display: none;">
            <div class="card bg-light border-primary mb-4">
              <div class="card-header bg-primary text-white fw-bold d-flex justify-content-between align-items-center">
                <span><i class="fa-solid fa-receipt me-2"></i>Huling Order (Last Order Details)</span>
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
                    <p class="mb-1 text-muted small fw-bold">ITEMS / PRODUCTS BOUGHT:</p>
                    <p class="fs-5 text-dark fw-semibold mb-0" id="lastOrderProducts">-</p>
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

            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-clock-rotate-left me-2"></i>Lahat ng Naging Transaksyon ni Customer (Complete History)</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark">
                  <tr>
                    <th>Date</th>
                    <th>Location</th>
                    <th>Products</th>
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
            <h6 class="fw-bold text-primary mb-2">Dedicated Products Inventory Sheet (VMC White, Busco, Bais, Balayan, Crystal, Passi, GB, Dark, Casa, Baron, Cali, Matling, RD, SW, King, GW, Farola, Asin, Countess, CS, Polaris, Lard Big, Marg Big, Small Marg, I, II, III, Harina, CF)</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark text-center">
                  <tr>
                    <th class="text-start">Product Name</th>
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
              
              <!-- PER-DAY FILTER TOGGLE / DATE SELECTOR -->
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
              <input type="text" id="stdPayNotes" class="form-control" placeholder="e.g., Partial payment sa lumang utang">
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
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
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
            <button type="submit" class="btn btn-dark"><i class="fa-solid fa-floppy-disk me-1"></i>Update Adjustment</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- Modal para sa Pagdaragdag ng Bagong Produkto -->
  <div class="modal fade" id="addProductModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-box-open me-2"></i>Add Product / Stock-In (Pili Supplier)</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="addProductForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date (Petsa ng Stock-In):</label>
              <input type="date" id="newProdDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name:</label>
              <input type="text" id="newProdName" class="form-control" placeholder="e.g., Semento" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Supplier / Galing Kay:</label>
              <input type="text" id="newProdSupplier" class="form-control" placeholder="e.g. Supplier A, ABC Trading" required>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Cost / Unit (₱):</label>
                <input type="number" step="0.01" id="newProdCost" class="form-control" placeholder="0.00" value="0.00" required>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Price / Unit (₱):</label>
                <input type="number" step="0.01" id="newProdPrice" class="form-control" placeholder="0.00" value="0.00" required>
              </div>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Quantity (Dami):</label>
              <input type="number" id="newProdStock" class="form-control" value="1" min="1" required>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success"><i class="fa-solid fa-floppy-disk me-1"></i>Save Stock In</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: ITEM RETURN / ISAULI -->
  <div class="modal fade" id="returnModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header bg-info text-white">
          <h5 class="modal-title"><i class="fa-solid fa-rotate-left me-2"></i>Item Return / Isauli sa Inventory</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="returnForm">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Date (Petsa ng Return):</label>
              <input type="date" id="returnDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name (Pangalan ng Item):</label>
              <input type="text" id="returnProductName" class="form-control" placeholder="e.g. Semento" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Quantity na Isinauli (Qty):</label>
              <input type="number" id="returnQty" class="form-control" value="1" min="1" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Uri ng Return / Dahilan:</label>
              <select id="returnType" class="form-select">
                <option value="Customer Return">Customer Return (Ibinalik ng Customer - Nadagdag sa Stock)</option>
                <option value="Supplier Replacement">Supplier Replacement (Pinadala ng Supplier)</option>
              </select>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Notes / Customer Name:</label>
              <input type="text" id="returnNotes" class="form-control" placeholder="e.g., Sobrang kuha ni Juan">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-info text-white fw-bold"><i class="fa-solid fa-check me-1"></i>I-save ang Return</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    // System Users Database (Admin username & password set to 'admin')
    let defaultUsers = [
      { id: 1, name: "System Administrator", username: "admin", password: "admin", role: "Admin" },
      { id: 2, name: "Juan Cashier", username: "cashier", password: "password", role: "Staff" }
    ];

    let users = JSON.parse(localStorage.getItem('rmv_users'));
    if (!users || !users.some(u => u.username === 'admin')) {
      users = defaultUsers;
      localStorage.setItem('rmv_users', JSON.stringify(users));
    }

    let currentUser = JSON.parse(localStorage.getItem('rmv_current_user')) || null;

    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let inventory = JSON.parse(localStorage.getItem('rmv_inventory'));
    
    const defaultDedicatedNames = [
      "VMC White", "Busco", "Bais", "Balayan", "Crystal", "Passi", "GB", "Dark", "Casa", "Baron", "Cali", "Matling", "RD", "SW", "King", "GW", "Farola", "Asin", "Countess", "CS", "Polaris", "Lard Big", "Marg Big", "Small Marg", "I", "II", "III", "Harina", "CF", "Polaris"
    ];

    if (!inventory) {
      inventory = [
        { name: "Palm", cost: 0, price: 0, beginning: 0, stockIn: 0, ending: 0, category: "palmcoco" },
        { name: "Coco", cost: 0, price: 0, beginning: 0, stockIn: 0, ending: 0, category: "palmcoco" }
      ];
      defaultDedicatedNames.forEach(name => {
        inventory.push({ name: name, cost: 0, price: 0, beginning: 0, stockIn: 0, ending: 0, category: "dedicated" });
      });
      localStorage.setItem('rmv_inventory', JSON.stringify(inventory));
    } else {
      inventory.forEach(item => {
        if (!item.category) {
          const lowerName = item.name.toLowerCase();
          if (lowerName.includes('palm') || lowerName.includes('coco')) {
            item.category = 'palmcoco';
          } else {
            item.category = 'dedicated';
          }
        }
      });
      defaultDedicatedNames.forEach(defName => {
        if (!inventory.some(i => i.name.toLowerCase() === defName.toLowerCase())) {
          inventory.push({ name: defName, cost: 0, price: 0, beginning: 0, stockIn: 0, ending: 0, category: "dedicated" });
        }
      });
    }

    let stockInHistory = JSON.parse(localStorage.getItem('rmv_stockInHistory')) || [];
    let returnHistory = JSON.parse(localStorage.getItem('rmv_returnHistory')) || [];
    let bossAdjustments = JSON.parse(localStorage.getItem('rmv_bossAdjustments')) || [];
    let monthlyExpensesData = JSON.parse(localStorage.getItem('rmv_monthlyExpensesData')) || {};
    let cashBreakdownData = JSON.parse(localStorage.getItem('rmv_cashBreakdownData')) || {};
    let standalonePayments = JSON.parse(localStorage.getItem('rmv_standalonePayments')) || [];

    function getTodayDateString() {
      const now = new Date();
      const year = now.getFullYear();
      const month = String(now.getMonth() + 1).padStart(2, '0');
      const day = String(now.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    }

    let todayFormatted = getTodayDateString();
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;
    document.getElementById('bossDate').value = todayFormatted;
    document.getElementById('newProdDate').value = todayFormatted;
    document.getElementById('payDate').value = todayFormatted;
    document.getElementById('inventorySheetDate').value = todayFormatted;
    document.getElementById('returnDate').value = todayFormatted;
    document.getElementById('standaloneExpenseDate').value = todayFormatted;
    document.getElementById('standaloneBossDate').value = todayFormatted;
    document.getElementById('stdPayDate').value = todayFormatted;
    
    const nowObj = new Date();
    const currentMonthStr = `${nowObj.getFullYear()}-${String(nowObj.getMonth() + 1).padStart(2, '0')}`;
    document.getElementById('auditMonth').value = currentMonthStr;
    document.getElementById('standaloneExpenseMonth').value = currentMonthStr;
    document.getElementById('standaloneBossMonth').value = currentMonthStr;

    window.onload = function() {
      addPosRow();
     
      if (currentUser) {
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('currentUserName').innerText = `${currentUser.name} (${currentUser.role})`;
        const adminElements = document.querySelectorAll('.admin-only');
        adminElements.forEach(el => {
          el.style.display = currentUser.role === 'Admin' ? 'block' : 'none';
        });
      }

      loadMoneyBreakdown();
      generateDailyReport();
      renderCreditTable();
      renderStandalonePayments();
      renderInventoryTables();
      renderStockInHistory();
      renderCustomerSalesLog();
      renderDailyInventorySheet();
      renderExpensesTable();
      renderStandaloneExpensesLedger();
      renderStandaloneBossLedger();
    };

    function saveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_inventory', JSON.stringify(inventory));
      localStorage.setItem('rmv_stockInHistory', JSON.stringify(stockInHistory));
      localStorage.setItem('rmv_returnHistory', JSON.stringify(returnHistory));
      localStorage.setItem('rmv_bossAdjustments', JSON.stringify(bossAdjustments));
      localStorage.setItem('rmv_monthlyExpensesData', JSON.stringify(monthlyExpensesData));
      localStorage.setItem('rmv_cashBreakdownData', JSON.stringify(cashBreakdownData));
      localStorage.setItem('rmv_standalonePayments', JSON.stringify(standalonePayments));
      localStorage.setItem('rmv_users', JSON.stringify(users));
      if (currentUser) {
        localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
      } else {
        localStorage.removeItem('rmv_current_user');
      }
    }

    function manualSaveData() {
      saveData();
      alert('Tagumpay na na-save ang lahat ng data sa Local Storage!');
    }

    function handleEnterNext(event, currentInput) {
      if (event.key === 'Enter') {
        event.preventDefault();
        const inputs = Array.from(document.querySelectorAll('.denom-fund, .denom-count, .denom-coins'));
        const currentIndex = inputs.indexOf(currentInput);
        if (currentIndex > -1 && currentIndex < inputs.length - 1) {
          inputs[currentIndex + 1].focus();
          inputs[currentIndex + 1].select();
        }
      }
    }

    // ================= AUTHENTICATION LOGIC =================
    document.getElementById('loginForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const u = document.getElementById('loginUsername').value.trim();
      const p = document.getElementById('loginPassword').value.trim();

      const found = users.find(user => user.username === u && user.password === p);

      if (found) {
        currentUser = found;
        saveData();
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('loginError').classList.add('d-none');
        document.getElementById('currentUserName').innerText = `${currentUser.name} (${currentUser.role})`;
       
        const adminElements = document.querySelectorAll('.admin-only');
        adminElements.forEach(el => {
          el.style.display = currentUser.role === 'Admin' ? 'block' : 'none';
        });

        this.reset();
      } else {
        document.getElementById('loginError').classList.remove('d-none');
      }
    });

    function logout() {
      currentUser = null;
      localStorage.removeItem('rmv_current_user');
      document.getElementById('loginOverlay').style.display = 'flex';
    }

    function openChangeProfileModal() {
      if(!currentUser) return;
      document.getElementById('profDisplayName').value = currentUser.name;
      document.getElementById('profPassword').value = '';
      new bootstrap.Modal(document.getElementById('changeProfileModal')).show();
    }

    document.getElementById('changeProfileForm').addEventListener('submit', function(e) {
      e.preventDefault();
      currentUser.name = document.getElementById('profDisplayName').value;
      const newPass = document.getElementById('profPassword').value;
      if(newPass) currentUser.password = newPass;

      const uIndex = users.findIndex(u => u.id === currentUser.id);
      if(uIndex > -1) users[uIndex] = currentUser;

      saveData();
      document.getElementById('currentUserName').innerText = `${currentUser.name} (${currentUser.role})`;
      alert('Profile successfully updated!');
      bootstrap.Modal.getInstance(document.getElementById('changeProfileModal')).hide();
    });

    function openUserManagementModal() {
      renderUserList();
      new bootstrap.Modal(document.getElementById('userManagementModal')).show();
    }

    function renderUserList() {
      const tbody = document.getElementById('userListBody');
      tbody.innerHTML = '';
      users.forEach((u, index) => {
        tbody.innerHTML += `
          <tr>
            <td>${u.name}</td>
            <td><code>${u.username}</code></td>
            <td><span class="badge ${u.role === 'Admin' ? 'bg-danger' : 'bg-secondary'}">${u.role}</span></td>
            <td class="text-center">
              ${u.id !== 1 ? `<button class="btn btn-sm btn-outline-danger border-0 p-0" onclick="deleteUser(${index})"><i class="fa-solid fa-trash"></i></button>` : `<small class="text-muted">Master</small>`}
            </td>
          </tr>
        `;
      });
    }

    document.getElementById('newUserForm').addEventListener('submit', function(e) {
      e.preventDefault();
      users.push({
        id: Date.now(),
        name: document.getElementById('newAccName').value,
        username: document.getElementById('newAccUser').value,
        password: document.getElementById('newAccPass').value,
        role: document.getElementById('newAccRole').value
      });
      saveData();
      this.reset();
      renderUserList();
    });

    function deleteUser(index) {
      if(confirm('Sigurado ka bang gusto mong alisin ang user na ito?')) {
        users.splice(index, 1);
        saveData();
        renderUserList();
      }
    }

    // ================= POS & INVENTORY ONLY LOGIC =================
    function toggleInventoryOnlyMode() {
      const isInventoryOnly = document.getElementById('inventoryOnlyMode').checked;
      const priceCols = document.querySelectorAll('.price-col');
      const containerBox = document.getElementById('containerSectionBox');
      const financialBox = document.getElementById('financialSectionBox');
      const creditSection = document.getElementById('creditFieldsSection');
      const locationSelect = document.getElementById('transactionLocation');

      priceCols.forEach(col => col.style.display = isInventoryOnly ? 'none' : '');
      containerBox.style.display = isInventoryOnly ? 'none' : 'block';
      financialBox.style.display = isInventoryOnly ? 'none' : 'flex';
      creditSection.style.display = 'none';

      if(isInventoryOnly) {
        locationSelect.value = 'Inventory Only';
        document.getElementById('totalAmount').value = '0.00';
      } else {
        locationSelect.value = 'Hiway';
      }

      const rows = document.querySelectorAll('#posItemsBody tr');
      rows.forEach(row => {
        const costInput = row.querySelector('.pos-cost').closest('td');
        const priceInput = row.querySelector('.pos-price').closest('td');
        const subtotalInput = row.querySelector('.pos-subtotal').closest('td');

        costInput.style.display = isInventoryOnly ? 'none' : '';
        priceInput.style.display = isInventoryOnly ? 'none' : '';
        subtotalInput.style.display = isInventoryOnly ? 'none' : '';
      });
    }

    function addPosRow() {
      const isInventoryOnly = document.getElementById('inventoryOnlyMode').checked;
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now() + Math.random().toString(36).substring(2, 5);
     
      const displayStyle = isInventoryOnly ? 'style="display: none;"' : '';

      const rowHTML = `
        <tr id="row-${rowId}">
          <td>
            <input type="text" class="form-control form-control-sm pos-name" placeholder="Pangalan ng Produkto" required>
          </td>
          <td>
            <input type="text" class="form-control form-control-sm pos-desc" placeholder="Description / Specification">
          </td>
          <td><input type="number" class="form-control form-control-sm pos-qty" value="1" min="1" oninput="calculateTotal()" required></td>
          <td ${displayStyle}><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" oninput="calculateTotal()" ${isInventoryOnly ? '' : 'required'}></td>
          <td ${displayStyle}><input type="number" step="0.01" class="form-control form-control-sm pos-price" placeholder="0.00" oninput="calculateTotal()" ${isInventoryOnly ? '' : 'required'}></td>
          <td ${displayStyle}><input type="number" step="0.01" class="form-control form-control-sm bg-light pos-subtotal" placeholder="0.00" readonly></td>
          <td class="col-action no-print">
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
        alert('Kailangang mayroong kahit isang produkto.');
      }
    }

    function toggleContainerFields() {
      const status = document.getElementById('containerStatus').value;
      const qtyGroup = document.querySelector('.container-qty-group');
      const depositGroup = document.querySelector('.container-deposit-group');

      if (status === 'HIRAM') {
        qtyGroup.style.display = 'block';
        depositGroup.style.display = 'none';
        document.getElementById('containerDepositRate').value = '0';
      } else if (status === 'DEPOSIT') {
        qtyGroup.style.display = 'block';
        depositGroup.style.display = 'block';
      } else {
        qtyGroup.style.display = 'none';
        depositGroup.style.display = 'none';
        document.getElementById('containerDepositRate').value = '0';
      }
      calculateTotal();
    }

    function calculateTotal() {
      const isInventoryOnly = document.getElementById('inventoryOnlyMode').checked;
      if (isInventoryOnly) return;

      const rows = document.querySelectorAll('#posItemsBody tr');
      let grandTotal = 0;
      rows.forEach(row => {
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const price = parseFloat(row.querySelector('.pos-price').value) || 0;
        const subtotal = qty * price;
        row.querySelector('.pos-subtotal').value = subtotal.toFixed(2);
        grandTotal += subtotal;
      });

      const status = document.getElementById('containerStatus').value;
      if (status === 'DEPOSIT') {
        const cQty = parseFloat(document.getElementById('containerQty').value) || 0;
        const cRate = parseFloat(document.getElementById('containerDepositRate').value) || 0;
        grandTotal += (cQty * cRate);
      }

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
      const isInventoryOnly = document.getElementById('inventoryOnlyMode').checked;

      const saleDate = document.getElementById('saleDate').value;
      const custName = document.getElementById('customerName').value;
      const location = document.getElementById('transactionLocation').value;

      let total = 0;
      let paid = 0;
      let balance = 0;
      let status = "PAID";
      let method = "Inventory Update";

      if (!isInventoryOnly) {
        total = parseFloat(document.getElementById('totalAmount').value) || 0;
        const paymentType = document.getElementById('paymentType').value;
        method = document.getElementById('paymentMethod').value;
        paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
       
        if (paymentType === 'FULL') paid = total;
        if (paymentType === 'CREDIT') paid = 0;

        balance = total - paid;
        if (balance > 0 && paid > 0) status = "PARTIAL";
        if (balance > 0 && paid === 0) status = "UNPAID";
      }

      const itemRows = document.querySelectorAll('#posItemsBody tr');
      let productSummary = [];
      let itemsPurchasedList = [];
      let totalCostOfGoods = 0;

      itemRows.forEach(row => {
        const name = row.querySelector('.pos-name').value.trim();
        const desc = row.querySelector('.pos-desc').value.trim();
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = isInventoryOnly ? 0 : (parseFloat(row.querySelector('.pos-cost').value) || 0);
        const price = isInventoryOnly ? 0 : (parseFloat(row.querySelector('.pos-price').value) || 0);

        totalCostOfGoods += (qty * cost);
        if(name) {
          let itemString = desc ? `${name} (${desc}) (x${qty})` : `${name} (x${qty})`;
          productSummary.push(itemString);
          itemsPurchasedList.push({
            name: name,
            desc: desc,
            qty: qty,
            cost: cost,
            price: price,
            subtotal: qty * price,
            location: location,
            date: saleDate,
            customer: custName
          });

          const invItem = inventory.find(inv => inv.name.toLowerCase() === name.toLowerCase());
          if(invItem) {
            if (isInventoryOnly) {
              invItem.ending += qty;
              invItem.stockIn += qty;
            } else {
              invItem.ending = Math.max(0, invItem.ending - qty);
            }
          } else {
            const lowerN = name.toLowerCase();
            const cat = (lowerN.includes('palm') || lowerN.includes('coco')) ? 'palmcoco' : 'dedicated';
            inventory.push({
              name: name,
              cost: cost,
              price: price,
              beginning: isInventoryOnly ? qty : 0,
              stockIn: isInventoryOnly ? qty : 0,
              ending: qty,
              category: cat
            });
          }

          if (isInventoryOnly) {
            stockInHistory.push({
              date: saleDate,
              product: name,
              qty: qty,
              supplier: 'Direct Inventory Only',
              note: `Manual Entry / ${custName}`
            });
          }
        }
      });

      let cInfo = "Wala";
      if (!isInventoryOnly) {
        const cStatus = document.getElementById('containerStatus').value;
        const cQty = document.getElementById('containerQty').value || 0;
        const cRate = parseFloat(document.getElementById('containerDepositRate').value) || 0;

        if (cStatus === 'HIRAM') {
          cInfo = `Hiram (${cQty} pcs)`;
        } else if (cStatus === 'DEPOSIT') {
          cInfo = `May Deposito (${cQty} pcs - ₱${(cQty * cRate).toFixed(2)})`;
        }
      }

      const paymentHistory = [];
      if (paid > 0) {
        paymentHistory.push({ amount: paid, method: method, date: saleDate });
      }

      transactions.push({
        id: Date.now(),
        date: saleDate,
        customer: custName,
        location: location,
        product: productSummary.join(', '),
        itemsList: itemsPurchasedList,
        containerInfo: cInfo,
        total: total,
        totalCost: totalCostOfGoods,
        netProfit: total - totalCostOfGoods,
        paid: paid,
        balance: balance,
        dueDate: isInventoryOnly ? 'N/A' : (document.getElementById('dueDate').value || 'N/A'),
        status: status,
        payments: paymentHistory
      });

      saveData();
      alert(isInventoryOnly ? 'Tagumpay na naidagdag sa Inventory!' : 'Transaction saved successfully!');
      this.reset();
      document.getElementById('posItemsBody').innerHTML = '';
      document.getElementById('inventoryOnlyMode').checked = false;
      toggleInventoryOnlyMode();
      addPosRow();
      document.getElementById('saleDate').value = getTodayDateString();
      renderInventoryTables();
      renderDailyInventorySheet();
    });

    // ================= ADD PRODUCT / STOCK IN =================
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const prodDate = document.getElementById('newProdDate').value || getTodayDateString();
      const name = document.getElementById('newProdName').value.trim();
      const supplier = document.getElementById('newProdSupplier').value.trim() || 'General Supplier';
      const cost = parseFloat(document.getElementById('newProdCost').value) || 0;
      const price = parseFloat(document.getElementById('newProdPrice').value) || 0;
      const qty = parseInt(document.getElementById('newProdStock').value) || 0;

      let existing = inventory.find(i => i.name.toLowerCase() === name.toLowerCase());
      if(existing) {
        existing.cost = cost;
        existing.price = price;
        existing.stockIn += qty;
        existing.ending += qty;
      } else {
        const lowerN = name.toLowerCase();
        const cat = (lowerN.includes('palm') || lowerN.includes('coco')) ? 'palmcoco' : 'dedicated';
        inventory.push({
          name: name,
          cost: cost,
          price: price,
          beginning: qty,
          stockIn: 0,
          ending: qty,
          category: cat
        });
      }

      if(qty > 0) {
        stockInHistory.push({
          date: prodDate,
          product: name,
          qty: qty,
          supplier: supplier,
          note: existing ? 'Nagdagdag ng Stock' : 'Bagong Produkto / Beginning Stock'
        });
      }

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      this.reset();
      document.getElementById('newProdDate').value = getTodayDateString();
      renderInventoryTables();
      renderStockInHistory();
      renderDailyInventorySheet();
      alert('Tagumpay na naidagdag ang produkto at nailagay sa talaan kasama ang supplier!');
    });

    // ================= ITEM RETURN / ISAULI LOGIC =================
    document.getElementById('returnForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const rDate = document.getElementById('returnDate').value || getTodayDateString();
      const rName = document.getElementById('returnProductName').value.trim();
      const rQty = parseInt(document.getElementById('returnQty').value) || 0;
      const rType = document.getElementById('returnType').value;
      const rNotes = document.getElementById('returnNotes').value.trim();

      if (rQty <= 0) {
        alert('Ilagay ang tamang quantity.');
        return;
      }

      let invItem = inventory.find(i => i.name.toLowerCase() === rName.toLowerCase());
      if (invItem) {
        invItem.ending += rQty; 
      } else {
        const lowerN = rName.toLowerCase();
        const cat = (lowerN.includes('palm') || lowerN.includes('coco')) ? 'palmcoco' : 'dedicated';
        inventory.push({
          name: rName,
          cost: 0,
          price: 0,
          beginning: 0,
          stockIn: rQty,
          ending: rQty,
          category: cat
        });
      }

      returnHistory.push({
        date: rDate,
        product: rName,
        qty: rQty,
        type: rType,
        notes: rNotes
      });

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('returnModal')).hide();
      this.reset();
      document.getElementById('returnDate').value = getTodayDateString();
      renderInventoryTables();
      renderDailyInventorySheet();
      alert('Tagumpay na naisailalim sa Return at nadagdag ulit sa inventory stock!');
    });

    function renderInventoryTables() {
      const palmCocoTbody = document.getElementById('palmCocoInventoryTableBody');
      const dedicatedTbody = document.getElementById('dedicatedInventoryTableBody');
      const tfoot = document.getElementById('inventoryTableFooter');
      
      palmCocoTbody.innerHTML = '';
      dedicatedTbody.innerHTML = '';

      let totalCostVal = 0;
      let totalInventoryValue = 0;

      inventory.forEach((item, index) => {
        let sold = (item.beginning + item.stockIn) - item.ending;
        if (sold < 0) sold = 0;

        totalCostVal += (item.ending * item.cost);
        totalInventoryValue += (item.ending * item.price);

        const rowHTML = `
          <tr>
            <td><input type="text" class="form-control form-control-sm fw-bold" value="${item.name}" onchange="updateInventoryItem(${index}, 'name', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-center" value="${item.cost}" onchange="updateInventoryItem(${index}, 'cost', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm text-center" value="${item.price}" onchange="updateInventoryItem(${index}, 'price', this.value)"></td>
            <td class="text-center">${item.beginning}</td>
            <td class="text-center text-success fw-bold">+${item.stockIn}</td>
            <td class="text-center text-danger">${sold}</td>
            <td><input type="number" class="form-control form-control-sm text-center fw-bold text-primary inventory-input mx-auto" value="${item.ending}" onchange="updateInventoryItem(${index}, 'ending', this.value)"></td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteInventoryItem(${index})" title="Tanggalin ang Produkto">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;

        if (item.category === 'palmcoco') {
          palmCocoTbody.insertAdjacentHTML('beforeend', rowHTML);
        } else {
          dedicatedTbody.insertAdjacentHTML('beforeend', rowHTML);
        }
      });

      if (palmCocoTbody.children.length === 0) {
        palmCocoTbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Wala pang nakatalang Palm & Coco.</td></tr>`;
      }
      if (dedicatedTbody.children.length === 0) {
        dedicatedTbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Wala pang nakatalang dedicated products.</td></tr>`;
      }

      tfoot.innerHTML = `
        <tr>
          <td colspan="6" class="text-end">Total Inventory Valuation (Puhunan / Presyo):</td>
          <td colspan="2" class="text-start text-primary">₱${totalCostVal.toFixed(2)} (Cost) / ₱${totalInventoryValue.toFixed(2)} (SRP)</td>
        </tr>
      `;
      renderCustomerSalesLog();
      renderDailyInventorySheet();
    }

    function updateInventoryItem(index, field, value) {
      if (field === 'name') {
        inventory[index].name = value.trim();
        const lowerN = inventory[index].name.toLowerCase();
        if (lowerN.includes('palm') || lowerN.includes('coco')) {
          inventory[index].category = 'palmcoco';
        } else {
          inventory[index].category = 'dedicated';
        }
      } else {
        inventory[index][field] = parseFloat(value) || 0;
      }
      saveData();
      renderInventoryTables();
    }

    function deleteInventoryItem(index) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang produktong ito sa inventory?')) {
        inventory.splice(index, 1);
        saveData();
        renderInventoryTables();
        alert('Naalis na sa inventory ang produkto.');
      }
    }

    function renderStockInHistory() {
      const tbody = document.getElementById('stockInHistoryBody');
      const searchQuery = document.getElementById('searchStockInInput') ? document.getElementById('searchStockInInput'].value.toLowerCase() : '';
      tbody.innerHTML = '';

      const filtered = stockInHistory.filter(item =>
        item.product.toLowerCase().includes(searchQuery) || item.date.includes(searchQuery) || (item.supplier && item.supplier.toLowerCase().includes(searchQuery))
      );

      filtered.slice().reverse().forEach(item => {
        tbody.innerHTML += `
          <tr>
            <td>${item.date}</td>
            <td class="fw-bold">${item.product}</td>
            <td class="text-center text-success fw-bold">+${item.qty}</td>
            <td><span class="badge bg-secondary">${item.supplier || 'N/A'}</span></td>
            <td><span class="badge bg-info text-dark">${item.note}</span></td>
          </tr>
        `;
      });

      if(filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="5" class="text-center text-muted py-3">Walang nakitang tala ng pagdadagdag ng produkto.</td></tr>`;
      }
    }

    function renderCustomerSalesLog() {
      const tbody = document.getElementById('customerSalesLogBody');
      if (!tbody) return;
      tbody.innerHTML = '';

      let logs = [];
      transactions.forEach(t => {
        if (t.itemsList && t.itemsList.length > 0) {
          t.itemsList.forEach(item => {
            logs.push({
              date: t.date,
              customer: t.customer,
              location: t.location,
              product: item.name + (item.desc ? ` (${item.desc})` : ''),
              qty: item.qty,
              cost: item.cost,
              price: item.price,
              total: item.subtotal
            });
          });
        }
      });

      logs.slice().reverse().forEach(log => {
        tbody.innerHTML += `
          <tr>
            <td>${log.date}</td>
            <td class="fw-bold">${log.customer}</td>
            <td><span class="badge bg-secondary">${log.location}</span></td>
            <td>${log.product}</td>
            <td class="text-center">${log.qty}</td>
            <td class="text-end">₱${log.cost.toFixed(2)}</td>
            <td class="text-end">₱${log.price.toFixed(2)}</td>
            <td class="text-end fw-bold text-success">₱${log.total.toFixed(2)}</td>
          </tr>
        `;
      });

      if (logs.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Wala pang naitalang benta sa mga customer.</td></tr>`;
      }
    }

    // ================= PER-DAY INVENTORY SHEET =================
    function renderDailyInventorySheet() {
      const selectedDate = document.getElementById('inventorySheetDate').value || getTodayDateString();
      const palmCocoBody = document.getElementById('dailyPalmCocoSheetBody');
      const dedicatedBody = document.getElementById('dailyDedicatedSheetBody');
      if (!palmCocoBody || !dedicatedBody) return;

      palmCocoBody.innerHTML = '';
      dedicatedBody.innerHTML = '';

      inventory.forEach(item => {
        let soldToday = 0;
        let stockInToday = 0;
        let returnToday = 0;

        transactions.forEach(t => {
          if (t.date === selectedDate && t.itemsList) {
            t.itemsList.forEach(i => {
              if (i.name.toLowerCase() === item.name.toLowerCase()) {
                soldToday += i.qty;
              }
            });
          }
        });

        stockInHistory.forEach(s => {
          if (s.date === selectedDate && s.product.toLowerCase() === item.name.toLowerCase()) {
            stockInToday += s.qty;
          }
        });

        returnHistory.forEach(r => {
          if (r.date === selectedDate && r.product.toLowerCase() === item.name.toLowerCase()) {
            returnToday += r.qty;
          }
        });

        let currentTotalEnding = Math.max(0, item.ending);
        let beginningToday = currentTotalEnding + soldToday - stockInToday - returnToday;
        if (beginningToday < 0) beginningToday = 0;

        let endingToday = beginningToday + stockInToday + returnToday - soldToday;
        if (endingToday < 0) endingToday = 0;

        const rowHTML = `
          <tr>
            <td class="fw-bold">${item.name}</td>
            <td class="text-center fw-semibold text-secondary">${beginningToday}</td>
            <td class="text-center text-success fw-bold">+${stockInToday}</td>
            <td class="text-center text-info fw-bold">+${returnToday}</td>
            <td class="text-center text-danger fw-bold">-${soldToday}</td>
            <td class="text-center fw-bold text-success table-success fs-6">${endingToday}</td>
          </tr>
        `;

        if (item.category === 'palmcoco') {
          palmCocoBody.insertAdjacentHTML('beforeend', rowHTML);
        } else {
          dedicatedBody.insertAdjacentHTML('beforeend', rowHTML);
        }
      });
    }

    // ================= STANDALONE UTANG & PAYMENTS LEDGER LOGIC =================
    document.getElementById('standalonePaymentForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const pDate = document.getElementById('stdPayDate').value || todayFormatted;
      const pCust = document.getElementById('stdPayCustomer').value.trim();
      const pAmt = parseFloat(document.getElementById('stdPayAmount').value) || 0;
      const pMethod = document.getElementById('stdPayMethod').value;
      const pNotes = document.getElementById('stdPayNotes').value.trim() || 'Manual Collection';

      if (pAmt <= 0) {
        alert('Maglagay ng tamang halaga ng bayad.');
        return;
      }

      standalonePayments.push({
        date: pDate,
        customer: pCust,
        amount: pAmt,
        method: pMethod,
        notes: pNotes
      });

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('standalonePaymentModal')).hide();
      this.reset();
      document.getElementById('stdPayDate').value = getTodayDateString();
      renderStandalonePayments();
      generateDailyReport();
      alert('Tagumpay na naidagdag ang manual payment sa ledger at nailess sa collection!');
    });

    function renderStandalonePayments() {
      const tbody = document.getElementById('standalonePaymentTableBody');
      const tfoot = document.getElementById('standalonePaymentTableFooter');
      const searchQuery = document.getElementById('searchStandalonePaymentInput') ? document.getElementById('searchStandalonePaymentInput').value.toLowerCase() : '';
      if (!tbody) return;

      tbody.innerHTML = '';
      let totalPaidSum = 0;
      let countVisible = 0;

      standalonePayments.forEach((p, index) => {
        const rowText = `${p.date} ${p.customer} ${p.method} ${p.notes}`.toLowerCase();
        if (searchQuery && !rowText.includes(searchQuery)) return;

        countVisible++;
        totalPaidSum += p.amount;

        tbody.innerHTML += `
          <tr>
            <td>${p.date}</td>
            <td class="fw-bold">${p.customer}</td>
            <td><span class="badge bg-secondary">${p.method}</span></td>
            <td class="text-end text-success fw-bold">₱${p.amount.toFixed(2)}</td>
            <td>${p.notes}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteStandalonePayment(${index})" title="Delete Payment">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;
      });

      if (countVisible === 0) {
        tbody.innerHTML = `<tr><td colspan="6" class="text-center text-muted py-3">Wala pang nakitang manual payment o collection.</td></tr>`;
        tfoot.innerHTML = '';
      } else {
        tfoot.innerHTML = `
          <tr>
            <td colspan="3" class="text-end">KABUUANG MANUAL PAYMENTS / COLLECTIONS:</td>
            <td class="text-end text-success fw-bold">₱${totalPaidSum.toFixed(2)}</td>
            <td colspan="2" class="no-print"></td>
          </tr>
        `;
      }
    }

    function deleteStandalonePayment(index) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang manual payment na ito?')) {
        standalonePayments.splice(index, 1);
        saveData();
        renderStandalonePayments();
        generateDailyReport();
        alert('Naalis na ang payment record.');
      }
    }

    // ================= STANDALONE SALARY & EXPENSES STANDALONE LEDGER LOGIC =================
    function toggleExpenseViewMode() {
      const mode = document.getElementById('expenseViewMode').value;
      const monthInput = document.getElementById('standaloneExpenseMonth');
      const dateInput = document.getElementById('standaloneExpenseDate');
      if (mode === 'day') {
        monthInput.style.display = 'none';
        dateInput.style.display = 'block';
      } else {
        monthInput.style.display = 'block';
        dateInput.style.display = 'none';
      }
      renderStandaloneExpensesLedger();
    }

    function addStandaloneExpenseRow() {
      const mode = document.getElementById('expenseViewMode').value;
      const targetDate = mode === 'day' ? (document.getElementById('standaloneExpenseDate').value || todayFormatted) : ((document.getElementById('standaloneExpenseMonth').value || currentMonthStr) + '-01');
      const targetMonthKey = targetDate.substring(0, 7);

      if (!monthlyExpensesData[targetMonthKey]) {
        monthlyExpensesData[targetMonthKey] = [];
      }
      monthlyExpensesData[targetMonthKey].push({
        date: targetDate, // Auto date batay sa napiling petsa o ngayon
        salaryName: '',
        salaryAmount: 0,
        expenseName: '',
        expenseAmount: 0
      });
      saveData();
      renderStandaloneExpensesLedger();
      renderExpensesTable();
      generateMonthlyAudit();
    }

    function renderStandaloneExpensesLedger() {
      const mode = document.getElementById('expenseViewMode').value;
      const targetMonthKey = document.getElementById('standaloneExpenseMonth').value || currentMonthStr;
      const targetDateKey = document.getElementById('standaloneExpenseDate').value || todayFormatted;
      
      const tbody = document.getElementById('standaloneExpenseTableBody');
      const tfoot = document.getElementById('standaloneExpenseTableFooter');
      const searchQuery = document.getElementById('searchStandaloneExpenseInput') ? document.getElementById('searchStandaloneExpenseInput').value.toLowerCase() : '';
      
      tbody.innerHTML = '';

      if (!monthlyExpensesData[targetMonthKey]) {
        monthlyExpensesData[targetMonthKey] = [];
      }

      const expensesList = monthlyExpensesData[targetMonthKey];
      let hasVisibleRow = false;
      let totalSalarySum = 0;
      let totalExpenseSum = 0;

      expensesList.forEach((item, index) => {
        const itemDate = item.date || (targetMonthKey + '-01');
        
        // Kapag naka-day view, ipapakita lamang ang mga nakatala sa eksaktong araw na iyon
        if (mode === 'day' && itemDate !== targetDateKey) {
          return;
        }

        const rowText = `${itemDate} ${item.salaryName}${item.salaryAmount} ${item.expenseName}${item.expenseAmount}`.toLowerCase();
        if (searchQuery && !rowText.includes(searchQuery)) {
          return;
        }
        hasVisibleRow = true;

        totalSalarySum += (parseFloat(item.salaryAmount) || 0);
        totalExpenseSum += (parseFloat(item.expenseAmount) || 0);

        tbody.innerHTML += `
          <tr>
            <td>
              <input type="date" class="form-control form-control-sm" value="${itemDate}" onchange="updateStandaloneExpenseField(${index}, 'date', this.value)">
            </td>
            <td>
              <input type="text" class="form-control form-control-sm" placeholder="e.g. Sahod ni Juan" value="${item.salaryName || ''}" onchange="updateStandaloneExpenseField(${index}, 'salaryName', this.value)">
            </td>
            <td>
              <input type="number" step="0.01" class="form-control form-control-sm text-end" placeholder="0.00" value="${item.salaryAmount || 0}" onchange="updateStandaloneExpenseField(${index}, 'salaryAmount', this.value)">
            </td>
            <td>
              <input type="text" class="form-control form-control-sm" placeholder="e.g. Kuryente / Tubig / Bigas" value="${item.expenseName || ''}" onchange="updateStandaloneExpenseField(${index}, 'expenseName', this.value)">
            </td>
            <td>
              <input type="number" step="0.01" class="form-control form-control-sm text-end" placeholder="0.00" value="${item.expenseAmount || 0}" onchange="updateStandaloneExpenseField(${index}, 'expenseAmount', this.value)">
            </td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteStandaloneExpenseRow(${index})">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;
      });

      if (!hasVisibleRow) {
        tbody.innerHTML = `<tr><td colspan="6" class="text-center text-muted py-3">Wala pang nakitang salary o expenses para sa araw na ito. Pindutin ang "Add Expense / Salary Line" para magdagdag ng bagong talaan para sa araw na ito.</td></tr>`;
      }

      let combinedTotal = totalSalarySum + totalExpenseSum;

      document.getElementById('totalSalarySumDisplay').innerText = `₱${totalSalarySum.toFixed(2)}`;
      document.getElementById('totalExpenseSumDisplay').innerText = `₱${totalExpenseSum.toFixed(2)}`;
      document.getElementById('totalCombinedExpenseDisplay').innerText = `₱${combinedTotal.toFixed(2)}`;

      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">KABUUANG SUBTOTAL / TOTAL:</td>
          <td class="text-end text-danger fw-bold">₱${totalSalarySum.toFixed(2)}</td>
          <td class="text-end"></td>
          <td class="text-end text-danger fw-bold">₱${totalExpenseSum.toFixed(2)}</td>
          <td class="no-print"></td>
        </tr>
      `;
    }

    function updateStandaloneExpenseField(index, field, value) {
      const selectedMonth = document.getElementById('standaloneExpenseMonth').value || currentMonthStr;
      if (field === 'salaryAmount' || field === 'expenseAmount') {
        monthlyExpensesData[selectedMonth][index][field] = parseFloat(value) || 0;
      } else {
        monthlyExpensesData[selectedMonth][index][field] = value;
      }
      saveData();
      renderStandaloneExpensesLedger();
      renderExpensesTable();
      generateMonthlyAudit();
      generateDailyReport();
    }

    function deleteStandaloneExpenseRow(index) {
      const selectedMonth = document.getElementById('standaloneExpenseMonth').value || currentMonthStr;
      if (confirm('Sigurado ka bang gusto mong tanggalin ang entry na ito?')) {
        monthlyExpensesData[selectedMonth].splice(index, 1);
        saveData();
        renderStandaloneExpensesLedger();
        renderExpensesTable();
        generateMonthlyAudit();
        generateDailyReport();
      }
    }

    // ================= STANDALONE D/ECO BOSS LEDGER LOGIC =================
    function toggleBossViewMode() {
      const mode = document.getElementById('bossViewMode').value;
      const monthInput = document.getElementById('standaloneBossMonth');
      const dateInput = document.getElementById('standaloneBossDate');
      if (mode === 'day') {
        monthInput.style.display = 'none';
        dateInput.style.display = 'block';
      } else {
        monthInput.style.display = 'block';
        dateInput.style.display = 'none';
      }
      renderStandaloneBossLedger();
    }

    function renderStandaloneBossLedger() {
      const mode = document.getElementById('bossViewMode').value;
      const selectedMonth = document.getElementById('standaloneBossMonth').value || currentMonthStr;
      const selectedDate = document.getElementById('standaloneBossDate').value || todayFormatted;

      const tbody = document.getElementById('standaloneBossTableBody');
      const tfoot = document.getElementById('standaloneBossTableFooter');
      const searchQuery = document.getElementById('searchStandaloneBossInput') ? document.getElementById('searchStandaloneBossInput').value.toLowerCase() : '';
      
      tbody.innerHTML = '';

      let totalAdd = 0;
      let totalSub = 0;
      let countVisible = 0;

      bossAdjustments.forEach((b, originalIndex) => {
        if (b.date.startsWith(selectedMonth)) {
          if (mode === 'day' && b.date !== selectedDate) return;

          let rowText = `${b.date} ${b.type} ${b.amount} ${b.notes}`.toLowerCase();
          if (searchQuery && !rowText.includes(searchQuery)) return;

          countVisible++;
          let val = parseFloat(b.amount) || 0;
          let actionButtons = `
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary border-0 p-1 me-1" onclick="openEditBossModal(${originalIndex})" title="Edit Adjustment">
                <i class="fa-solid fa-pen-to-square"></i>
              </button>
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteBossAdjustment(${originalIndex})" title="Delete Adjustment">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          `;

          if (b.type === 'ADD') {
            totalAdd += val;
            tbody.innerHTML += `<tr><td>${b.date}</td><td><span class="badge bg-success">Boss Addition</span> ${b.notes}</td><td class="text-end text-success">+₱${val.toFixed(2)}</td>${actionButtons}</tr>`;
          } else {
            totalSub += val;
            tbody.innerHTML += `<tr><td>${b.date}</td><td><span class="badge bg-danger">Boss Withdrawal</span> ${b.notes}</td><td class="text-end text-danger">-₱${val.toFixed(2)}</td>${actionButtons}</tr>`;
          }
        }
      });

      if (countVisible === 0) {
        tbody.innerHTML = `<tr><td colspan="4" class="text-center text-muted py-3">Wala pang nakitang D/Eco Boss adjustments sa panahong ito.</td></tr>`;
        tfoot.innerHTML = '';
      } else {
        let netBoss = totalAdd - totalSub;
        document.getElementById('totalBossAddDisplay').innerText = `+₱${totalAdd.toFixed(2)}`;
        document.getElementById('totalBossSubDisplay').innerText = `-₱${totalSub.toFixed(2)}`;
        document.getElementById('totalBossNetDisplay').innerText = `₱${netBoss.toFixed(2)}`;

        tfoot.innerHTML = `
          <tr>
            <td colspan="2" class="text-end">KABUUANG SUBTOTAL (Additions: <span class="text-success">+₱${totalAdd.toFixed(2)}</span> | Withdrawals: <span class="text-danger">-₱${totalSub.toFixed(2)}</span>):</td>
            <td class="text-end fw-bold ${netBoss >= 0 ? 'text-success' : 'text-danger'}">₱${netBoss.toFixed(2)}</td>
            <td class="no-print"></td>
          </tr>
        `;
      }
    }

    // ================= DAILY REPORT & MONEY BREAKDOWN LOGIC =================
    let currentTargetCashInDrawer = 0;
    let currentDayDebtPayments = 0;
    let currentDayExpenses = 0;

    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';

      let daySales = 0;
      let dayHiwaySales = 0;
      let dayByaheSales = 0;
      let dayHiwayGrossProfit = 0;
      let dayByaheGrossProfit = 0;

      let dayCollected = 0;
      let dayGrossProfit = 0;
      let count = 0;

      let totalByaheCash = 0;
      let totalGCash = 0;
      let totalBT = 0;
      let totalCheque = 0;
      let dayDebtPayments = 0;
      let dayTotalCreditBalance = 0;

      const filtered = transactions.filter(t => t.date === selectedDate || (t.payments && t.payments.some(p => p.date === selectedDate)));

      filtered.forEach((t, index) => {
        const txCost = t.totalCost || 0;
        const netProf = t.total - txCost;

        if(t.date === selectedDate) {
          daySales += t.total;
          dayGrossProfit += netProf;

          if (t.location === 'Hiway') {
            dayHiwaySales += t.total;
            dayHiwayGrossProfit += netProf;
          } else if (t.location === 'Byahe') {
            dayByaheSales += t.total;
            dayByaheGrossProfit += netProf;
          }

          // Kunin ang natitirang balance/utang para sa transaksyong ginawa sa araw na ito
          dayTotalCreditBalance += (parseFloat(t.balance) || 0);
        }
       
        if (t.payments) {
          t.payments.forEach((p, pIdx) => {
            if (p.date === selectedDate) {
              dayCollected += p.amount;
              if (t.date !== selectedDate || pIdx > 0) {
                dayDebtPayments += p.amount;
              }

              if (p.method === 'Byahe Cash') totalByaheCash += p.amount;
              else if (p.method === 'GCash') totalGCash += p.amount;
              else if (p.method === 'Bank Transfer' || p.method === 'BT') totalBT += p.amount;
              else if (p.method === 'Cheque') totalCheque += p.amount;
            }
          });
        }
       
        count++;

        const lastMethod = (t.payments && t.payments.length > 0) ? t.payments[t.payments.length - 1].method : 'N/A';
        let locBadge = '<span class="badge bg-primary">Hiway</span>';
        if (t.location === 'Byahe') locBadge = '<span class="badge bg-info text-dark">Byahe</span>';
        if (t.location === 'Inventory Only') locBadge = '<span class="badge bg-secondary">Inventory Only</span>';

        tbody.innerHTML += `
          <tr>
            <td>${index + 1}</td>
            <td class="fw-bold">${t.customer}</td>
            <td>${locBadge}</td>
            <td>${t.product}</td>
            <td><span class="badge bg-secondary">${t.containerInfo || 'Wala'}</span></td>
            <td class="text-secondary fw-semibold">₱${txCost.toFixed(2)}</td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger">₱${t.balance.toFixed(2)}</td>
            <td class="text-success fw-bold">₱${netProf.toFixed(2)}</td>
            <td>${lastMethod}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary border-0 p-1" onclick="openEditTransactionModal(${t.id})" title="Edit Transaction & Cost">
                <i class="fa-solid fa-pen-to-square"></i>
              </button>
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteTransaction(${t.id})" title="Delete Transaction">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;
      });

      // Isama rin ang mga nakatalang standalone manual payments para sa napiling petsa sa koleksyon at breakdown
      if (standalonePayments) {
        standalonePayments.forEach(p => {
          if (p.date === selectedDate) {
            dayCollected += p.amount;
            dayDebtPayments += p.amount;
            if (p.method === 'Byahe Cash') totalByaheCash += p.amount;
            else if (p.method === 'GCash') totalGCash += p.amount;
            else if (p.method === 'Bank Transfer' || p.method === 'BT') totalBT += p.amount;
            else if (p.method === 'Cheque') totalCheque += p.amount;
          }
        });
      }

      currentDayDebtPayments = dayDebtPayments;

      let dayExpensesTotal = 0;
      const auditMonthStr = selectedDate.substring(0, 7);
      if (monthlyExpensesData && monthlyExpensesData[auditMonthStr]) {
        monthlyExpensesData[auditMonthStr].forEach(exp => {
          if (!exp.date || exp.date === selectedDate || exp.date.startsWith(selectedDate)) {
            dayExpensesTotal += (parseFloat(exp.salaryAmount) || 0) + (parseFloat(exp.expenseAmount) || 0);
          }
        });
      }
      currentDayExpenses = dayExpensesTotal;

      let dayHiwayNet = dayHiwayGrossProfit;
      let dayByaheNet = dayByaheGrossProfit;
      let daySubtotalNet = dayHiwayNet + dayByaheNet;
      let dayNetProfit = daySubtotalNet - dayExpensesTotal;

      let totalNonCashToday = totalByaheCash + totalGCash + totalBT + totalCheque;
      let cashSalesToday = daySales - totalNonCashToday; 
      
      const fundInputVal = parseFloat(document.getElementById('cashFundInput').value) || 0;
      
      // Target Cash in Drawer = Cash Sales - Utang/Balance + Payment sa lumang utang - Expenses
      currentTargetCashInDrawer = Math.max(0, cashSalesToday - dayTotalCreditBalance + dayDebtPayments - dayExpensesTotal);

      document.getElementById('dailyHiwaySales').innerText = `₱${dayHiwaySales.toFixed(2)}`;
      document.getElementById('dailyHiwayProfit').innerText = `₱${dayHiwayNet.toFixed(2)}`;
      document.getElementById('dailyByaheSales').innerText = `₱${dayByaheSales.toFixed(2)}`;
      document.getElementById('dailyByaheProfit').innerText = `₱${dayByaheNet.toFixed(2)}`;

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTotalNetProfit').innerText = `₱${dayNetProfit.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;

      document.getElementById('totalCollectionAll').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('breakdownDebtPayment').innerText = `+₱${dayDebtPayments.toFixed(2)}`;
      document.getElementById('lessByaheCash').innerText = `-₱${totalByaheCash.toFixed(2)}`;
      document.getElementById('lessGCash').innerText = `-₱${totalGCash.toFixed(2)}`;
      document.getElementById('lessBT').innerText = `-₱${totalBT.toFixed(2)}`;
      document.getElementById('lessCheque').innerText = `-₱${totalCheque.toFixed(2)}`;
      document.getElementById('lessExpenses').innerText = `-₱${dayExpensesTotal.toFixed(2)}`;
      document.getElementById('lessCreditBalance').innerText = `-₱${dayTotalCreditBalance.toFixed(2)}`;
      document.getElementById('breakdownTargetSales').innerText = `₱${currentTargetCashInDrawer.toFixed(2)}`;

      loadMoneyBreakdown();
    }

    function calculateMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const fundInputVal = parseFloat(document.getElementById('cashFundInput').value) || 0;

      let totalCountedRaw = 0;
      let totalPcs = 0;

      const counts = document.querySelectorAll('.denom-count');
      const subtotals = document.querySelectorAll('.denom-subtotal');

      let breakdownObj = { fund: fundInputVal, counts: {} };

      counts.forEach((input, index) => {
        const denom = parseFloat(input.getAttribute('data-denom'));
        const qty = parseInt(input.value) || 0;
        const sub = qty * denom;
        subtotals[index].value = sub.toFixed(2);
        totalCountedRaw += sub;
        totalPcs += qty;
        breakdownObj.counts[denom] = qty;
      });

      const coinsInput = document.querySelector('.denom-coins');
      const coinsVal = parseFloat(coinsInput.value) || 0;
      totalCountedRaw += coinsVal;
      breakdownObj.coins = coinsVal;

      cashBreakdownData[selectedDate] = breakdownObj;
      saveData();

      document.getElementById('breakdownTotalPcs').innerText = `${totalPcs} pcs`;
      document.getElementById('breakdownTotalAmount').innerText = `₱${totalCountedRaw.toFixed(2)}`;

      const targetWithFund = currentTargetCashInDrawer + fundInputVal;
      document.getElementById('breakdownTargetWithFund').innerText = `₱${targetWithFund.toFixed(2)}`;

      const totalCountedSalesOnly = Math.max(0, totalCountedRaw - fundInputVal);
      document.getElementById('totalCountedCash').innerText = `₱${totalCountedSalesOnly.toFixed(2)}`;

      const discrepancy = totalCountedSalesOnly - currentTargetCashInDrawer;
      const discEl = document.getElementById('cashDiscrepancy');
      const alertEl = document.getElementById('cashStatusAlert');

      discEl.innerText = `₱${discrepancy.toFixed(2)}`;

      if (Math.abs(discrepancy) < 0.01) {
        discEl.className = "fs-5 fw-bold text-success";
        alertEl.className = "alert alert-success text-center p-2 fw-bold mb-0";
        alertEl.innerHTML = `<i class="fa-solid fa-circle-check me-1"></i> BALANSE ANG CASH! Walang kulang o sobra.`;
      } else if (discrepancy > 0) {
        discEl.className = "fs-5 fw-bold text-primary";
        alertEl.className = "alert alert-primary text-center p-2 fw-bold mb-0";
        alertEl.innerHTML = `<i class="fa-solid fa-arrow-up me-1"></i> OVER SA CASH (Sobra nang ₱${discrepancy.toFixed(2)})`;
      } else {
        discEl.className = "fs-5 fw-bold text-danger";
        alertEl.className = "alert alert-danger text-center p-2 fw-bold mb-0";
        alertEl.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> SHORT SA CASH (Kulang nang ₱${Math.abs(discrepancy).toFixed(2)})`;
      }
    }

    function clearMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      delete cashBreakdownData[selectedDate];
      saveData();

      document.getElementById('cashFundInput').value = '';
      document.querySelectorAll('.denom-count').forEach(inp => inp.value = '');
      document.querySelectorAll('.denom-subtotal').forEach(inp => inp.value = '0.00');
      document.querySelector('.denom-coins').value = '';
      calculateMoneyBreakdown();
    }

    function loadMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const data = cashBreakdownData[selectedDate];

      if (data) {
        document.getElementById('cashFundInput').value = data.fund || '';
        document.querySelector('.denom-coins').value = data.coins || '';

        const counts = document.querySelectorAll('.denom-count');
        counts.forEach(input => {
          const denom = input.getAttribute('data-denom');
          if (data.counts && data.counts[denom] !== undefined) {
            input.value = data.counts[denom];
          } else {
            input.value = '';
          }
        });
      } else {
        document.getElementById('cashFundInput').value = '';
        document.querySelectorAll('.denom-count').forEach(inp => inp.value = '');
        document.querySelector('.denom-coins').value = '';
      }
      calculateMoneyBreakdown();
    }

    // ================= UTANG & PAYMENTS LOGIC =================
    function renderCreditTable() {
      const tbody = document.getElementById('creditTableBody');
      const paidTbody = document.getElementById('paidHistoryTableBody');
      tbody.innerHTML = '';
      paidTbody.innerHTML = '';

      let creditCount = 0;
      let paidCount = 0;

      transactions.forEach(t => {
        const currentBalance = Number((t.balance || 0).toFixed(2));
        const currentPaid = Number((t.paid || 0).toFixed(2));
        const currentTotal = Number((t.total || 0).toFixed(2));

        if (currentBalance > 0.01) {
          creditCount++;
          let locBadge = '<span class="badge bg-primary">Hiway</span>';
          if (t.location === 'Byahe') locBadge = '<span class="badge bg-info text-dark">Byahe</span>';

          tbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td>${locBadge}</td>
              <td>${t.product}</td>
              <td class="text-secondary fw-semibold">₱${(t.totalCost || 0).toFixed(2)}</td>
              <td class="text-success">₱${currentPaid.toFixed(2)}</td>
              <td class="text-danger fw-bold">₱${currentBalance.toFixed(2)}</td>
              <td>${t.dueDate || 'N/A'}</td>
              <td><span class="badge bg-warning text-dark">${t.status || 'PARTIAL'}</span></td>
              <td class="no-print">
                <button class="btn btn-sm btn-success" onclick="openPaymentModal(${t.id})">
                  <i class="fa-solid fa-peso-sign me-1"></i> Magbayad
                </button>
              </td>
            </tr>
          `;
        } else if (currentTotal > 0 || currentPaid > 0) {
          paidCount++;
          paidTbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td>${t.product}</td>
              <td>₱${currentTotal.toFixed(2)}</td>
              <td class="text-success">₱${currentPaid.toFixed(2)}</td>
              <td><span class="badge bg-success">PAID / FULL</span></td>
              <td class="text-center no-print">
                <button class="btn btn-sm btn-outline-primary border-0 p-1" onclick="openPaymentModal(${t.id})" title="Tingnan ang Payment History">
                  <i class="fa-solid fa-eye"></i>
                </button>
              </td>
            </tr>
          `;
        }
      });

      if (creditCount === 0) {
        tbody.innerHTML = `<tr><td colspan="9" class="text-center text-muted py-3">Walang aktibong utang sa kasalukuyan.</td></tr>`;
      }
      if (paidCount === 0) {
        paidTbody.innerHTML = `<tr><td colspan="6" class="text-center text-muted py-3">Wala pang nakatalang bayad na transaksyon.</td></tr>`;
      }
    }

    function openPaymentModal(txId) {
      const t = transactions.find(item => item.id === txId);
      if(!t) return;

      document.getElementById('payTxId').value = t.id;
      document.getElementById('payCustomerName').value = t.customer;
      document.getElementById('payTotalAmount').value = `₱${t.total.toFixed(2)}`;
      document.getElementById('payRemainingBalance').value = `₱${t.balance.toFixed(2)}`;
      document.getElementById('payAmountNow').value = t.balance > 0 ? t.balance.toFixed(2) : '0.00';
      document.getElementById('payMethod').value = 'Cash';
      document.getElementById('payDate').value = getTodayDateString();

      new bootstrap.Modal(document.getElementById('paymentModal')).show();
    }

    document.getElementById('paymentForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const txId = parseInt(document.getElementById('payTxId').value);
      const t = transactions.find(item => item.id === txId);
      if (!t) return;

      const payAmt = parseFloat(document.getElementById('payAmountNow').value) || 0;
      const payMethod = document.getElementById('payMethod').value;
      const payDate = document.getElementById('payDate').value || getTodayDateString();

      if (payAmt <= 0) {
        alert('Mangyaring maglagay ng tamang halaga ng bayad.');
        return;
      }
      if (payAmt > t.balance + 0.01) {
        alert('Ang ibinigay na bayad ay mas malaki kaysa sa natitirang balanse!');
        return;
      }

      t.paid = Number((t.paid + payAmt).toFixed(2));
      t.balance = Number(Math.max(0, t.total - t.paid).toFixed(2));
      
      if (t.balance <= 0.01) {
        t.balance = 0;
        t.status = 'PAID';
      } else {
        t.status = 'PARTIAL';
      }

      if (!t.payments) t.payments = [];
      t.payments.push({ amount: payAmt, method: payMethod, date: payDate });

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('paymentModal')).hide();
      renderCreditTable();
      searchCustomerOrder();
      generateDailyReport();
      alert('Tagumpay na naitala ang pagbabayad!');
    });

    // ================= CUSTOMER ORDER LOOKUP =================
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

        const last = matched[matched.length - 1];
        document.getElementById('lastOrderCustomer').innerText = last.customer;
        document.getElementById('lastOrderDate').innerText = last.date;
        document.getElementById('lastOrderContainer').innerText = last.containerInfo || 'Wala';
        document.getElementById('lastOrderProducts').innerText = last.product;
        document.getElementById('lastOrderTotal').innerText = `₱${last.total.toFixed(2)}`;
        document.getElementById('lastOrderPaid').innerText = `₱${last.paid.toFixed(2)}`;
        document.getElementById('lastOrderBalance').innerText = `₱${last.balance.toFixed(2)}`;

        const badge = document.getElementById('lastOrderBadge');
        if (last.balance === 0) {
          badge.className = "badge bg-success fs-6";
          badge.innerText = "PAID";
        } else if (last.paid > 0) {
          badge.className = "badge bg-warning text-dark fs-6";
          badge.innerText = "PARTIAL";
        } else {
          badge.className = "badge bg-danger fs-6";
          badge.innerText = "UNPAID / CREDIT";
        }

        const historyBody = document.getElementById('customerHistoryBody');
        historyBody.innerHTML = '';
        matched.slice().reverse().forEach(item => {
          let sBadge = '<span class="badge bg-success">PAID</span>';
          if (item.balance > 0 && item.paid > 0) sBadge = '<span class="badge bg-warning text-dark">PARTIAL</span>';
          else if (item.balance > 0) sBadge = '<span class="badge bg-danger">UNPAID</span>';

          historyBody.innerHTML += `
            <tr>
              <td>${item.date}</td>
              <td><span class="badge bg-secondary">${item.location}</span></td>
              <td>${item.product}</td>
              <td>${item.containerInfo || 'Wala'}</td>
              <td>₱${item.total.toFixed(2)}</td>
              <td class="text-success">₱${item.paid.toFixed(2)}</td>
              <td class="text-danger">₱${item.balance.toFixed(2)}</td>
              <td class="text-success fw-bold">₱${(item.netProfit || (item.total - (item.totalCost || 0))).toFixed(2)}</td>
              <td>${sBadge}</td>
              <td class="text-center no-print">
                <button class="btn btn-sm btn-success py-0 px-2 me-1" onclick="openPaymentModal(${item.id})" title="Magbayad / Add Payment">
                  <i class="fa-solid fa-peso-sign"></i>
                </button>
                <button class="btn btn-sm btn-outline-primary border-0 p-1 me-1" onclick="openEditTransactionModal(${item.id})" title="Edit Transaction">
                  <i class="fa-solid fa-pen-to-square"></i>
                </button>
                <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteTransaction(${item.id}); searchCustomerOrder();" title="Delete Transaction">
                  <i class="fa-solid fa-trash-can"></i>
                </button>
              </td>
            </tr>
          `;
        });
      } else {
        container.style.display = 'none';
        notFound.classList.remove('d-none');
      }
    }

    // ================= MONTHLY AUDIT & EXPENSES LOGIC =================
    function addExpenseRow() {
      const auditMonth = document.getElementById('auditMonth').value;
      if (!monthlyExpensesData[auditMonth]) {
        monthlyExpensesData[auditMonth] = [];
      }
      monthlyExpensesData[auditMonth].push({
        date: todayFormatted, // Auto date sa kasalukuyang araw
        salaryName: '',
        salaryAmount: 0,
        expenseName: '',
        expenseAmount: 0
      });
      saveData();
      renderExpensesTable();
      renderStandaloneExpensesLedger();
      generateMonthlyAudit();
    }

    function renderExpensesTable() {
      const tbody = document.getElementById('expenseTableBody');
      const tfoot = document.getElementById('expenseTableFooter');
      const auditMonth = document.getElementById('auditMonth').value;
      const searchQuery = document.getElementById('searchExpenseInput') ? document.getElementById('searchExpenseInput').value.toLowerCase() : '';
      
      tbody.innerHTML = '';

      if (!monthlyExpensesData[auditMonth]) {
        monthlyExpensesData[auditMonth] = [];
      }

      const expensesList = monthlyExpensesData[auditMonth];
      let hasVisibleRow = false;
      let totalSalarySum = 0;
      let totalExpenseSum = 0;

      expensesList.forEach((item, index) => {
        const rowText = `${item.date} ${item.salaryName}${item.salaryAmount} ${item.expenseName}${item.expenseAmount}`.toLowerCase();
        if (searchQuery && !rowText.includes(searchQuery)) {
          return;
        }
        hasVisibleRow = true;

        totalSalarySum += (parseFloat(item.salaryAmount) || 0);
        totalExpenseSum += (parseFloat(item.expenseAmount) || 0);

        tbody.innerHTML += `
          <tr>
            <td>
              <input type="date" class="form-control form-control-sm" value="${item.date || todayFormatted}" onchange="updateExpenseField(${index}, 'date', this.value)">
            </td>
            <td>
              <input type="text" class="form-control form-control-sm" placeholder="e.g. Sahod ni Juan" value="${item.salaryName || ''}" onchange="updateExpenseField(${index}, 'salaryName', this.value)">
            </td>
            <td>
              <input type="number" step="0.01" class="form-control form-control-sm text-end" placeholder="0.00" value="${item.salaryAmount || 0}" onchange="updateExpenseField(${index}, 'salaryAmount', this.value)">
            </td>
            <td>
              <input type="text" class="form-control form-control-sm" placeholder="e.g. Kuryente / Tubig / Bigas" value="${item.expenseName || ''}" onchange="updateExpenseField(${index}, 'expenseName', this.value)">
            </td>
            <td>
              <input type="number" step="0.01" class="form-control form-control-sm text-end" placeholder="0.00" value="${item.expenseAmount || 0}" onchange="updateExpenseField(${index}, 'expenseAmount', this.value)">
            </td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteExpenseRow(${index})">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          </tr>
        `;
      });

      if (!hasVisibleRow) {
        tbody.innerHTML = `<tr><td colspan="6" class="text-center text-muted py-3">Walang natagpuang salary o expense para sa paghahanap na ito. Pindutin ang "Add Expense Line" para magdagdag.</td></tr>`;
      }

      tfoot.innerHTML = `
        <tr>
          <td colspan="2" class="text-end">SUBTOTAL / TOTAL:</td>
          <td class="text-end text-danger fw-bold">₱${totalSalarySum.toFixed(2)}</td>
          <td class="text-end"></td>
          <td class="text-end text-danger fw-bold">₱${totalExpenseSum.toFixed(2)}</td>
          <td class="no-print"></td>
        </tr>
      `;
    }

    function updateExpenseField(index, field, value) {
      const auditMonth = document.getElementById('auditMonth').value;
      if (field === 'salaryAmount' || field === 'expenseAmount') {
        monthlyExpensesData[auditMonth][index][field] = parseFloat(value) || 0;
      } else {
        monthlyExpensesData[auditMonth][index][field] = value;
      }
      saveData();
      renderExpensesTable();
      renderStandaloneExpensesLedger();
      generateMonthlyAudit();
      generateDailyReport();
    }

    function deleteExpenseRow(index) {
      const auditMonth = document.getElementById('auditMonth').value;
      if (confirm('Sigurado ka bang gusto mong tanggalin ang entry na ito?')) {
        monthlyExpensesData[auditMonth].splice(index, 1);
        saveData();
        renderExpensesTable();
        renderStandaloneExpensesLedger();
        generateMonthlyAudit();
        generateDailyReport();
      }
    }

    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('auditMonth').value; // YYYY-MM
      if (!selectedMonth) return;

      let totalSales = 0;
      let totalCost = 0;
      let totalExpenses = 0;

      let hiwaySales = 0;
      let hiwayCost = 0;
      let hiwayGrossProfit = 0;

      let byaheSales = 0;
      let byaheCost = 0;
      let byaheGrossProfit = 0;

      transactions.forEach(t => {
        if (t.date.startsWith(selectedMonth)) {
          const tCost = t.totalCost || 0;
          const tGross = t.total - tCost;
          totalSales += t.total;
          totalCost += tCost;

          if (t.location === 'Hiway') {
            hiwaySales += t.total;
            hiwayCost += tCost;
            hiwayGrossProfit += tGross;
          } else if (t.location === 'Byahe') {
            byaheSales += t.total;
            byaheCost += tCost;
            byaheGrossProfit += tGross;
          }
        }
      });

      if (monthlyExpensesData[selectedMonth]) {
        monthlyExpensesData[selectedMonth].forEach(exp => {
          totalExpenses += (parseFloat(exp.salaryAmount) || 0) + (parseFloat(exp.expenseAmount) || 0);
        });
      }

      let hiwayNet = hiwayGrossProfit;
      let byaheNet = byaheGrossProfit;
      
      let bossNetAdjustment = 0;
      let bossSubtotalAdd = 0;
      let bossSubtotalSub = 0;
      const bossBody = document.getElementById('bossLogsBody');
      const bossFooter = document.getElementById('bossLogsFooter');
      bossBody.innerHTML = '';
      
      const searchBossQuery = document.getElementById('searchBossInput') ? document.getElementById('searchBossInput').value.toLowerCase() : '';
      let filteredBossCount = 0;

      let bossByDate = {};
      bossAdjustments.forEach((b, originalIndex) => {
        if (b.date.startsWith(selectedMonth)) {
          let rowText = `${b.date} ${b.type} ${b.amount} ${b.notes}`.toLowerCase();
          if (searchBossQuery && !rowText.includes(searchQuery)) return;

          if (!bossByDate[b.date]) {
            bossByDate[b.date] = [];
          }
          bossByDate[b.date].push({ ...b, originalIndex });
        }
      });

      const sortedBossDates = Object.keys(bossByDate).sort().reverse();
      sortedBossDates.forEach(dateKey => {
        let dayAddTotal = 0;
        let daySubTotal = 0;

        bossByDate[dateKey].forEach(item => {
          filteredBossCount++;
          let val = parseFloat(item.amount) || 0;
          let actionButtons = `
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary border-0 p-1 me-1" onclick="openEditBossModal(${item.originalIndex})" title="Edit Adjustment">
                <i class="fa-solid fa-pen-to-square"></i>
              </button>
              <button class="btn btn-sm btn-outline-danger border-0 p-1" onclick="deleteBossAdjustment(${item.originalIndex})" title="Delete Adjustment">
                <i class="fa-solid fa-trash-can"></i>
              </button>
            </td>
          `;

          if (item.type === 'ADD') {
            bossNetAdjustment += val;
            bossSubtotalAdd += val;
            dayAddTotal += val;
            bossBody.innerHTML += `<tr><td>${item.date}</td><td><span class="badge bg-success">Boss Addition</span> ${item.notes}</td><td class="text-end text-success">+₱${val.toFixed(2)}</td>${actionButtons}</tr>`;
          } else {
            bossNetAdjustment -= val;
            bossSubtotalSub += val;
            daySubTotal += val;
            bossBody.innerHTML += `<tr><td>${item.date}</td><td><span class="badge bg-danger">Boss Withdrawal</span> ${item.notes}</td><td class="text-end text-danger">-₱${val.toFixed(2)}</td>${actionButtons}</tr>`;
          }
        });

        bossBody.innerHTML += `
          <tr class="table-light fw-semibold">
            <td colspan="2" class="text-end text-muted small">Subtotal para sa ${dateKey}:</td>
            <td class="text-end text-muted small">Add: +₱${dayAddTotal.toFixed(2)} | With: -₱${daySubTotal.toFixed(2)}</td>
            <td class="no-print"></td>
          </tr>
        `;
      });

      if (filteredBossCount === 0) {
        bossBody.innerHTML = `<tr><td colspan="4" class="text-center text-muted py-2">Wala pang D/Eco Boss adjustments na nahanap.</td></tr>`;
        bossFooter.innerHTML = '';
      } else {
        bossFooter.innerHTML = `
          <tr>
            <td colspan="2" class="text-end">KABUUANG SUBTOTAL (Additions: <span class="text-success">+₱${bossSubtotalAdd.toFixed(2)}</span> | Withdrawals: <span class="text-danger">-₱${bossSubtotalSub.toFixed(2)}</span>):</td>
            <td class="text-end fw-bold ${bossNetAdjustment >= 0 ? 'text-success' : 'text-danger'}">₱${bossNetAdjustment.toFixed(2)}</td>
            <td class="no-print"></td>
          </tr>
        `;
      }

      let subtotalNet = hiwayNet + byaheNet - totalExpenses + bossNetAdjustment;
      let netProfit = subtotalNet;

      document.getElementById('auditHiwaySales').innerText = `₱${hiwaySales.toFixed(2)}`;
      document.getElementById('auditHiwayCost').innerText = `₱${hiwayCost.toFixed(2)}`;
      document.getElementById('auditHiwayNetProfit').innerText = `₱${hiwayNet.toFixed(2)}`;

      document.getElementById('auditByaheSales').innerText = `₱${byaheSales.toFixed(2)}`;
      document.getElementById('auditByaheCost').innerText = `₱${byaheCost.toFixed(2)}`;
      document.getElementById('auditByaheNetProfit').innerText = `₱${byaheNet.toFixed(2)}`;

      document.getElementById('auditTotalSales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('auditTotalCost').innerText = `₱${totalCost.toFixed(2)}`;
      document.getElementById('auditGrossProfit').innerText = `₱${subtotalNet.toFixed(2)}`;
      document.getElementById('auditExpenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${netProfit.toFixed(2)}`;

      const dailyBreakdownBody = document.getElementById('auditDailyBreakdownBody');
      dailyBreakdownBody.innerHTML = '';

      let dailyMap = {};
      transactions.forEach(t => {
        if (t.date.startsWith(selectedMonth)) {
          if (!dailyMap[t.date]) {
            dailyMap[t.date] = { hiwayNet: 0, byaheNet: 0, cost: 0, subtotalNet: 0 };
          }
          const tGross = t.total - (t.totalCost || 0);
          if (t.location === 'Hiway') dailyMap[t.date].hiwayNet += tGross;
          if (t.location === 'Byahe') dailyMap[t.date].byaheNet += tGross;
          dailyMap[t.date].cost += (t.totalCost || 0);
          dailyMap[t.date].subtotalNet += (dailyMap[t.date].hiwayNet + dailyMap[t.date].byaheNet);
        }
      });

      const sortedDates = Object.keys(dailyMap).sort().reverse();
      sortedDates.forEach(d => {
        const item = dailyMap[d];
        
        let dayExpSum = 0;
        if (monthlyExpensesData[selectedMonth]) {
          monthlyExpensesData[selectedMonth].forEach(exp => {
            if (!exp.date || exp.date === d || exp.date.startsWith(d)) {
              dayExpSum += (parseFloat(exp.salaryAmount) || 0) + (parseFloat(exp.expenseAmount) || 0);
            }
          });
        }

        let dayBossSum = 0;
        bossAdjustments.forEach(b => {
          if (b.date === d) {
            if (b.type === 'ADD') dayBossSum += (parseFloat(b.amount) || 0);
            else dayBossSum -= (parseFloat(b.amount) || 0);
          }
        });

        const dayBaseSubNet = item.hiwayNet + item.byaheNet;
        const dayFinalNet = dayBaseSubNet - dayExpSum + dayBossSum;

        dailyBreakdownBody.innerHTML += `
          <tr>
            <td class="fw-bold">${d}</td>
            <td class="text-end">₱${item.hiwayNet.toFixed(2)}</td>
            <td class="text-end">₱${item.byaheNet.toFixed(2)}</td>
            <td class="text-end fw-semibold">₱${dayBaseSubNet.toFixed(2)}</td>
            <td class="text-end text-success fw-bold">₱${dayFinalNet.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary py-0 px-2" onclick="jumpToDailyReport('${d}')">
                <i class="fa-solid fa-eye me-1"></i> View
              </button>
            </td>
          </tr>
        `;
      });

      if (sortedDates.length === 0) {
        dailyBreakdownBody.innerHTML = `<tr><td colspan="6" class="text-center text-muted py-3">Wala pang nahanap na transaksyon sa buwang ito.</td></tr>`;
      }

      renderExpensesTable();
      renderStandaloneBossLedger();
    }

    function jumpToDailyReport(dateStr) {
      document.getElementById('dailyReportDate').value = dateStr;
      generateDailyReport();
      const dailyTabButton = document.getElementById('daily-tab');
      const tab = new bootstrap.Tab(dailyTabButton);
      tab.show();
    }

    document.getElementById('bossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      bossAdjustments.push({
        date: document.getElementById('bossDate').value,
        type: document.getElementById('bossType').value,
        amount: parseFloat(document.getElementById('bossAmount').value) || 0,
        notes: document.getElementById('bossNotes').value.trim()
      });
      saveData();
      bootstrap.Modal.getInstance(document.getElementById('bossModal')).hide();
      this.reset();
      document.getElementById('bossDate').value = getTodayDateString();
      generateMonthlyAudit();
      renderStandaloneBossLedger();
      alert('Tagumpay na naidagdag ang D/Eco Boss adjustment!');
    });

    function openEditBossModal(index) {
      const b = bossAdjustments[index];
      if (!b) return;

      document.getElementById('editBossIndex').value = index;
      document.getElementById('editBossDate').value = b.date;
      document.getElementById('editBossType').value = b.type;
      document.getElementById('editBossAmount').value = b.amount;
      document.getElementById('editBossNotes').value = b.notes;

      new bootstrap.Modal(document.getElementById('editBossModal')).show();
    }

    document.getElementById('editBossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const index = parseInt(document.getElementById('editBossIndex').value);
      if (index > -1 && bossAdjustments[index]) {
        bossAdjustments[index] = {
          date: document.getElementById('editBossDate').value,
          type: document.getElementById('editBossType').value,
          amount: parseFloat(document.getElementById('editBossAmount').value) || 0,
          notes: document.getElementById('editBossNotes').value.trim()
        };
        saveData();
        bootstrap.Modal.getInstance(document.getElementById('editBossModal')).hide();
        generateMonthlyAudit();
        renderStandaloneBossLedger();
        alert('Tagumpay na na-update ang D/Eco Boss adjustment!');
      }
    });

    function deleteBossAdjustment(index) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang adjustment na ito?')) {
        bossAdjustments.splice(index, 1);
        saveData();
        generateMonthlyAudit();
        renderStandaloneBossLedger();
        alert('Naalis na ang adjustment.');
      }
    }

    // ================= GLOBAL SEARCH LOGIC =================
    function renderGlobalSearchResults() {
      const query = document.getElementById('globalSearchInput').value.trim().toLowerCase();
      const wrapper = document.getElementById('globalSearchResultsWrapper');
      const tbody = document.getElementById('globalSearchResultsBody');
      tbody.innerHTML = '';

      if (!query) {
        wrapper.style.display = 'none';
        return;
      }

      wrapper.style.display = 'block';
      let results = [];

      transactions.forEach(t => {
        let matchStr = `${t.date} ${t.customer}${t.location} ${t.product}${t.status}`.toLowerCase();
        if (matchStr.includes(query)) {
          results.push({
            date: t.date,
            customer: t.customer,
            location: t.location,
            details: t.product,
            total: t.total,
            paid: t.paid,
            balance: t.balance,
            status: t.status,
            id: t.id,
            type: 'POS'
          });
        }
      });

      results.forEach(r => {
        tbody.innerHTML += `
          <tr>
            <td>${r.date}</td>
            <td class="fw-bold">${r.customer}</td>
            <td><span class="badge bg-secondary">${r.location}</span></td>
            <td>${r.details}</td>
            <td>₱${r.total.toFixed(2)}</td>
            <td class="text-success">₱${r.paid.toFixed(2)}</td>
            <td class="text-danger">₱${r.balance.toFixed(2)}</td>
            <td><span class="badge bg-info text-dark">${r.status}</span></td>
          </tr>
        `;
      });

      if (results.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Walang nakitang tugma sa iyong paghahanap.</td></tr>`;
      }
    }

    function clearGlobalSearch() {
      document.getElementById('globalSearchInput').value = '';
      document.getElementById('globalSearchResultsWrapper').style.display = 'none';
    }

    // ================= EDIT TRANSACTION MODAL LOGIC =================
    function openEditTransactionModal(txId) {
      const t = transactions.find(item => item.id === txId);
      if(!t) return;

      document.getElementById('editTxId').value = t.id;
      document.getElementById('editTxDate').value = t.date;
      document.getElementById('editCustomerName').value = t.customer;
      document.getElementById('editLocation').value = t.location;
      document.getElementById('editProduct').value = t.product;
      document.getElementById('editContainerInfo').value = t.containerInfo || '';
      document.getElementById('editTotalCost').value = t.totalCost || 0;
      document.getElementById('editTotal').value = t.total;
      document.getElementById('editPaid').value = t.paid;
      document.getElementById('editBalance').value = t.balance;

      new bootstrap.Modal(document.getElementById('editTransactionModal')).show();
    }

    function calculateEditBalance() {
      const total = parseFloat(document.getElementById('editTotal').value) || 0;
      const paid = parseFloat(document.getElementById('editPaid').value) || 0;
      const balance = Math.max(0, total - paid);
      document.getElementById('balance').value = balance.toFixed(2);
    }

    document.getElementById('editTransactionForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const txId = parseInt(document.getElementById('editTxId').value);
      const tIndex = transactions.findIndex(item => item.id === txId);

      if (tIndex > -1) {
        const newTotal = parseFloat(document.getElementById('editTotal').value) || 0;
        const newPaid = parseFloat(document.getElementById('editPaid').value) || 0;
        const newCost = parseFloat(document.getElementById('editTotalCost').value) || 0;
        const newBalance = Math.max(0, newTotal - newPaid);

        transactions[tIndex].date = document.getElementById('editTxDate').value;
        transactions[tIndex].customer = document.getElementById('editCustomerName').value;
        transactions[tIndex].location = document.getElementById('editLocation').value;
        transactions[tIndex].product = document.getElementById('editProduct').value;
        transactions[tIndex].containerInfo = document.getElementById('editContainerInfo').value;
        transactions[tIndex].totalCost = newCost;
        transactions[tIndex].netProfit = newTotal - newCost;
        transactions[tIndex].total = newTotal;
        transactions[tIndex].paid = newPaid;
        transactions[tIndex].balance = newBalance;
        transactions[tIndex].status = newBalance === 0 ? 'PAID' : (newPaid > 0 ? 'PARTIAL' : 'UNPAID');

        if (transactions[tIndex].payments && transactions[tIndex].payments.length > 0) {
          transactions[tIndex].payments[0].amount = newPaid;
          transactions[tIndex].payments[0].date = transactions[tIndex].date;
        } else if (newPaid > 0) {
          transactions[tIndex].payments = [{ amount: newPaid, method: 'Cash', date: transactions[tIndex].date }];
        }

        saveData();
        bootstrap.Modal.getInstance(document.getElementById('editTransactionModal')).hide();
        generateDailyReport();
        renderCreditTable();
        searchCustomerOrder();
        alert('Tagumpay na na-update ang transaksyon!');
      }
    });

    function deleteTransaction(txId) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang transaksyong ito?')) {
        transactions = transactions.filter(t => t.id !== txId);
        saveData();
        generateDailyReport();
        renderCreditTable();
        searchCustomerOrder();
        alert('Naalis na ang transaksyon.');
      }
    }
  </script>
</body>
</html>
