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
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTable(); renderStockInHistory(); renderCustomerSalesLog();">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item admin-only">
          <button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()">
            <i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit & Net Profit
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
                <span class="text-muted small fw-bold">DAILY TOTAL SALES</span>
                <h4 class="text-primary mt-1 mb-0" id="dailyTotalSales">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">TOTAL COLLECTION</span>
                <h4 class="text-success mt-1 mb-0" id="dailyTotalCollected">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;">
                <span class="text-muted small fw-bold">DAILY NET PROFIT</span>
                <h4 class="text-success fw-bold mt-1 mb-0" id="dailyTotalNetProfit">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #f57c00;">
                <span class="text-muted small fw-bold">TRANSACTIONS</span>
                <h4 class="text-warning mt-1 mb-0" id="dailyTxCount">0</h4>
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

                  <hr class="my-1">

                  <div class="d-flex justify-content-between align-items-center my-2 bg-light p-2 rounded">
                    <span class="fw-bold text-dark">Target Cash in Drawer (Sales):</span>
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
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> Print Utang List
            </button>
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
              <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#addProductModal">
                <i class="fa-solid fa-plus me-1"></i> Add Product
              </button>
            </div>
          </div>

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
              <tbody id="inventoryTableBody">
                <!-- Dynamic Content -->
              </tbody>
              <tfoot class="table-secondary fw-bold text-center" id="inventoryTableFooter">
                <!-- Total Row rendered dynamically -->
              </tfoot>
            </table>
          </div>

          <div class="card p-3 bg-light border mb-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="fw-bold text-secondary m-0"><i class="fa-solid fa-clock-rotate-left me-2"></i>Talaan kung kelan nagdadagdag ng Produkto (Stock-In History)</h5>
            </div>
            <div class="input-group mb-3">
              <span class="input-group-text bg-white"><i class="fa-solid fa-magnifying-glass"></i></span>
              <input type="text" id="searchStockInInput" class="form-control" placeholder="I-search ang pangalan ng produkto o petsa..." oninput="renderStockInHistory()">
            </div>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Petsa (Date Added)</th>
                    <th>Product Name</th>
                    <th class="text-center">Ibinagdag (Stock In Qty)</th>
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

      <!-- ================= 5. MONTHLY AUDIT TAB ================= -->
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
                <span class="text-muted small fw-bold">GROSS PROFIT</span>
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
                <span class="text-muted small fw-bold">NET PROFIT (Final Kita)</span>
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
                    <th class="text-end">Total Sales (₱)</th>
                    <th class="text-end">Total Cost / Puhunan (₱)</th>
                    <th class="text-end">Net Profit (₱)</th>
                    <th class="text-center no-print" style="width: 140px;">Action / Balikan</th>
                  </tr>
                </thead>
                <tbody id="auditDailyBreakdownBody">
                  <!-- Dynamic Daily Breakdown Rows -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- SALARY & EXPENSES BREAKDOWN (MAY HIWALAY NA COLUMN AT SEARCH BAR) -->
          <div class="card p-3 bg-light mb-4 border">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-receipt me-2"></i>Itemized Salary & Expenses Breakdown</h6>
              <button class="btn btn-sm btn-outline-danger" onclick="addExpenseRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Expense Line
              </button>
            </div>
            
            <!-- SEARCH BAR PARA SA EXPENSES AT SALARY -->
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
                <option value="ADD">Boss Addition / Capital Cash In (+ Net Profit)</option>
                <option value="SUB">Boss Withdrawal / Cash Out (- Net Profit)</option>
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
              <label class="form-label fw-semibold">Date (Petsa ng Stock-In):</label>
              <input type="date" id="newProdDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name:</label>
              <input type="text" id="newProdName" class="form-control" placeholder="e.g., Semento" required>
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
              <label class="form-label fw-semibold">Beginning Stock / Stock In Qty:</label>
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
    // System Users Database (Hardcoded default check to ensure admin/password works)
    let defaultUsers = [
      { id: 1, name: "System Administrator", username: "admin", password: "password", role: "Admin" },
      { id: 2, name: "Juan Cashier", username: "cashier", password: "password", role: "Staff" }
    ];

    let users = JSON.parse(localStorage.getItem('rmv_users'));
    if (!users || !users.some(u => u.username === 'admin')) {
      users = defaultUsers;
      localStorage.setItem('rmv_users', JSON.stringify(users));
    }

    let currentUser = JSON.parse(localStorage.getItem('rmv_current_user')) || null;

    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let inventory = JSON.parse(localStorage.getItem('rmv_inventory')) || [];
    let stockInHistory = JSON.parse(localStorage.getItem('rmv_stockInHistory')) || [];
    let bossAdjustments = JSON.parse(localStorage.getItem('rmv_bossAdjustments')) || [];
    let monthlyExpensesData = JSON.parse(localStorage.getItem('rmv_monthlyExpensesData')) || {};
    let cashBreakdownData = JSON.parse(localStorage.getItem('rmv_cashBreakdownData')) || {};

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
    
    const nowObj = new Date();
    document.getElementById('auditMonth').value = `${nowObj.getFullYear()}-${String(nowObj.getMonth() + 1).padStart(2, '0')}`;

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
      renderInventoryTable();
      renderExpensesTable();
    };

    function saveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_inventory', JSON.stringify(inventory));
      localStorage.setItem('rmv_stockInHistory', JSON.stringify(stockInHistory));
      localStorage.setItem('rmv_bossAdjustments', JSON.stringify(bossAdjustments));
      localStorage.setItem('rmv_monthlyExpensesData', JSON.stringify(monthlyExpensesData));
      localStorage.setItem('rmv_cashBreakdownData', JSON.stringify(cashBreakdownData));
      localStorage.setItem('rmv_users', JSON.stringify(users));
      if (currentUser) {
        localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
      } else {
        localStorage.removeItem('rmv_current_user');
      }
    }

    function manualSaveData() {
      saveData();
      alert('Tagumpay na nai-save ang lahat ng data sa Local Storage!');
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
        document.getElementById('loginError').classList.add('d-none');
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
            inventory.push({
              name: name,
              cost: cost,
              price: price,
              beginning: isInventoryOnly ? qty : 0,
              stockIn: isInventoryOnly ? qty : 0,
              ending: qty
            });
          }

          if (isInventoryOnly) {
            stockInHistory.push({
              date: saleDate,
              product: name,
              qty: qty,
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
    });

    // ================= ADD PRODUCT, EDIT & STOCK-IN INVENTORY LOGIC =================
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const prodDate = document.getElementById('newProdDate').value || getTodayDateString();
      const name = document.getElementById('newProdName').value.trim();
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
        inventory.push({
          name: name,
          cost: cost,
          price: price,
          beginning: qty,
          stockIn: 0,
          ending: qty
        });
      }

      if(qty > 0) {
        stockInHistory.push({
          date: prodDate,
          product: name,
          qty: qty,
          note: existing ? 'Nagdagdag ng Stock' : 'Bagong Produkto / Beginning Stock'
        });
      }

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      this.reset();
      document.getElementById('newProdDate').value = getTodayDateString();
      renderInventoryTable();
      renderStockInHistory();
      alert('Tagumpay na naidagdag ang produkto at nailagay sa talaan!');
    });

    function renderInventoryTable() {
      const tbody = document.getElementById('inventoryTableBody');
      const tfoot = document.getElementById('inventoryTableFooter');
      tbody.innerHTML = '';

      let totalCostVal = 0;
      let totalInventoryValue = 0;

      inventory.forEach((item, index) => {
        let sold = (item.beginning + item.stockIn) - item.ending;
        if (sold < 0) sold = 0;

        totalCostVal += (item.ending * item.cost);
        totalInventoryValue += (item.ending * item.price);

        tbody.innerHTML += `
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
      });

      if (inventory.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Wala pang nakatalang produkto sa inventory.</td></tr>`;
      }

      tfoot.innerHTML = `
        <tr>
          <td colspan="6" class="text-end">Total Inventory Valuation (Puhunan / Presyo):</td>
          <td colspan="2" class="text-start text-primary">₱${totalCostVal.toFixed(2)} (Cost) / ₱${totalInventoryValue.toFixed(2)} (SRP)</td>
        </tr>
      `;
      renderCustomerSalesLog();
    }

    function updateInventoryItem(index, field, value) {
      if (field === 'name') {
        inventory[index].name = value.trim();
      } else {
        inventory[index][field] = parseFloat(value) || 0;
      }
      saveData();
      renderInventoryTable();
    }

    function deleteInventoryItem(index) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang produktong ito sa inventory?')) {
        inventory.splice(index, 1);
        saveData();
        renderInventoryTable();
        alert('Naalis na sa inventory ang produkto.');
      }
    }

    function renderStockInHistory() {
      const tbody = document.getElementById('stockInHistoryBody');
      const searchQuery = document.getElementById('searchStockInInput') ? document.getElementById('searchStockInInput').value.toLowerCase() : '';
      tbody.innerHTML = '';

      const filtered = stockInHistory.filter(item =>
        item.product.toLowerCase().includes(searchQuery) || item.date.includes(searchQuery)
      );

      filtered.slice().reverse().forEach(item => {
        tbody.innerHTML += `
          <tr>
            <td>${item.date}</td>
            <td class="fw-bold">${item.product}</td>
            <td class="text-center text-success fw-bold">+${item.qty}</td>
            <td><span class="badge bg-info text-dark">${item.note}</span></td>
          </tr>
        `;
      });

      if(filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="4" class="text-center text-muted py-3">Walang nakitang tala ng pagdadagdag ng produkto.</td></tr>`;
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

    // ================= DAILY REPORT & MONEY BREAKDOWN LOGIC =================
    let currentTargetCashInDrawer = 0;
    let currentDayDebtPayments = 0;
    let currentDayExpenses = 0;

    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';

      let daySales = 0;
      let dayCollected = 0;
      let dayGrossProfit = 0;
      let count = 0;

      let totalGCash = 0;
      let totalBT = 0;
      let totalCheque = 0;
      let dayDebtPayments = 0;

      const filtered = transactions.filter(t => t.date === selectedDate || t.payments.some(p => p.date === selectedDate));

      filtered.forEach((t, index) => {
        const txCost = t.totalCost || 0;
        const netProf = t.total - txCost;

        if(t.date === selectedDate) {
          daySales += t.total;
          dayGrossProfit += netProf;
        }
       
        t.payments.forEach((p, pIdx) => {
          if (p.date === selectedDate) {
            dayCollected += p.amount;
            if (t.date !== selectedDate || pIdx > 0) {
              dayDebtPayments += p.amount;
            }

            if (p.method === 'GCash') totalGCash += p.amount;
            else if (p.method === 'Bank Transfer' || p.method === 'BT') totalBT += p.amount;
            else if (p.method === 'Cheque') totalCheque += p.amount;
          }
        });
       
        count++;

        const lastMethod = t.payments.length > 0 ? t.payments[t.payments.length - 1].method : 'N/A';
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

      if (filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="12" class="text-center text-muted py-3">Walang na-encode na transaksyon sa petsang ito.</td></tr>`;
      }

      currentDayDebtPayments = dayDebtPayments;

      // KUNIN ANG SALARY & EXPENSES NG PETSA NA ITO MULA SA MONTHLY EXPENSES DATA
      let dayExpensesTotal = 0;
      const auditMonthStr = selectedDate.substring(0, 7); // YYYY-MM
      if (monthlyExpensesData && monthlyExpensesData[auditMonthStr]) {
        monthlyExpensesData[auditMonthStr].forEach(exp => {
          if (!exp.date || exp.date === selectedDate || exp.date.startsWith(selectedDate)) {
            dayExpensesTotal += (parseFloat(exp.salaryAmount) || 0) + (parseFloat(exp.expenseAmount) || 0);
          }
        });
      }
      currentDayExpenses = dayExpensesTotal;

      let dayNetProfit = dayGrossProfit - dayExpensesTotal;

      // Automatic deduction of salary & expenses from cash collections
      currentTargetCashInDrawer = Math.max(0, dayCollected - (totalGCash + totalBT + totalCheque + dayExpensesTotal));

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTotalNetProfit').innerText = `₱${dayNetProfit.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;

      document.getElementById('totalCollectionAll').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('breakdownDebtPayment').innerText = `+₱${dayDebtPayments.toFixed(2)}`;
      document.getElementById('lessGCash').innerText = `-₱${totalGCash.toFixed(2)}`;
      document.getElementById('lessBT').innerText = `-₱${totalBT.toFixed(2)}`;
      document.getElementById('lessCheque').innerText = `-₱${totalCheque.toFixed(2)}`;
      document.getElementById('lessExpenses').innerText = `-₱${dayExpensesTotal.toFixed(2)}`;
      document.getElementById('breakdownTargetSales').innerText = `₱${currentTargetCashInDrawer.toFixed(2)}`;

      loadMoneyBreakdown();
    }

    function calculateMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const fundInputVal = parseFloat(document.getElementById('cashFundInput').value) || 0;

      let totalCountedRaw = 0; // Kabuuang pera sa drawer na binilang (kasama ang pondo)
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

      // I-display ang kabuuang pera sa drawer (kasama ang pondo) sa subtotal table
      document.getElementById('breakdownTotalPcs').innerText = `${totalPcs} pcs`;
      document.getElementById('breakdownTotalAmount').innerText = `₱${totalCountedRaw.toFixed(2)}`;

      const targetWithFund = currentTargetCashInDrawer + fundInputVal;
      document.getElementById('breakdownTargetWithFund').innerText = `₱${targetWithFund.toFixed(2)}`;

      // Bawasan ng pondo ang binu-biling total cash bago i-kumpara sa target sales upang hindi ito sumobra bilang benta
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
        if (t.balance > 0) {
          creditCount++;
          let locBadge = '<span class="badge bg-primary">Hiway</span>';
          if (t.location === 'Byahe') locBadge = '<span class="badge bg-info text-dark">Byahe</span>';

          tbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td>${locBadge}</td>
              <td>${t.product}</td>
              <td class="text-secondary fw-semibold">₱${(t.totalCost || 0).toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
              <td>${t.dueDate}</td>
              <td><span class="badge bg-warning text-dark">${t.status}</span></td>
              <td class="no-print">
                <button class="btn btn-sm btn-success" onclick="openPaymentModal(${t.id})">
                  <i class="fa-solid fa-peso-sign me-1"></i> Magbayad
                </button>
              </td>
            </tr>
          `;
        } else if (t.total > 0) {
          paidCount++;
          paidTbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td>${t.product}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
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

      let paymentHistoryHTML = '';
      if(t.payments && t.payments.length > 0) {
        t.payments.forEach(p => {
          paymentHistoryHTML += `<li>${p.date} - ₱${p.amount.toFixed(2)} (${p.method})</li>`;
        });
      } else {
        paymentHistoryHTML = `<li>Wala pang naitalang hulog.</li>`;
      }

      let promptMsg = `Customer: ${t.customer}\nTotal Amount: ₱${t.total.toFixed(2)}\nNaibayad Na: ₱${t.paid.toFixed(2)}\nNalalabing Utang (Balance): ₱${t.balance.toFixed(2)}\n\nMagkano ang idadagdag na bayad / hulog ngayon?`;
      let payAmtStr = prompt(promptMsg, t.balance);
      if(payAmtStr !== null) {
        let payAmt = parseFloat(payAmtStr) || 0;
        if(payAmt > 0) {
          if(payAmt > t.balance) {
            alert('Ang ibinigay na bayad ay mas malaki kaysa sa natitirang balanse!');
            return;
          }
          let payMethod = prompt('Ilagay ang Payment Method (Cash, GCash, Bank Transfer, Cheque):', 'Cash') || 'Cash';
          let payDate = prompt('Ilagay ang Petsa ng Pagbabayad (YYYY-MM-DD):', getTodayDateString()) || getTodayDateString();

          t.paid += payAmt;
          t.balance = Math.max(0, t.total - t.paid);
          if(t.balance === 0) t.status = 'PAID';
          else t.status = 'PARTIAL';

          if(!t.payments) t.payments = [];
          t.payments.push({ amount: payAmt, method: payMethod, date: payDate });

          saveData();
          renderCreditTable();
          alert('Tagumpay na naitala ang pagbabayad!');
        }
      }
    }

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
        date: auditMonth + '-01',
        salaryName: '',
        salaryAmount: 0,
        expenseName: '',
        expenseAmount: 0
      });
      saveData();
      renderExpensesTable();
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
              <input type="date" class="form-control form-control-sm" value="${item.date || auditMonth + '-01'}" onchange="updateExpenseField(${index}, 'date', this.value)">
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
      generateMonthlyAudit();
      generateDailyReport();
    }

    function deleteExpenseRow(index) {
      const auditMonth = document.getElementById('auditMonth').value;
      if (confirm('Sigurado ka bang gusto mong tanggalin ang entry na ito?')) {
        monthlyExpensesData[auditMonth].splice(index, 1);
        saveData();
        renderExpensesTable();
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

      transactions.forEach(t => {
        if (t.date.startsWith(selectedMonth)) {
          totalSales += t.total;
          totalCost += (t.totalCost || 0);
        }
      });

      if (monthlyExpensesData[selectedMonth]) {
        monthlyExpensesData[selectedMonth].forEach(exp => {
          totalExpenses += (parseFloat(exp.salaryAmount) || 0) + (parseFloat(exp.expenseAmount) || 0);
        });
      }

      let bossNetAdjustment = 0;
      const bossBody = document.getElementById('bossLogsBody');
      bossBody.innerHTML = '';
      bossAdjustments.forEach(b => {
        if (b.date.startsWith(selectedMonth)) {
          let val = parseFloat(b.amount) || 0;
          if (b.type === 'ADD') {
            bossNetAdjustment += val;
            bossBody.innerHTML += `<tr><td>${b.date}</td><td><span class="badge bg-success">Boss Addition</span> ${b.notes}</td><td class="text-success">+₱${val.toFixed(2)}</td></tr>`;
          } else {
            bossNetAdjustment -= val;
            bossBody.innerHTML += `<tr><td>${b.date}</td><td><span class="badge bg-danger">Boss Withdrawal</span> ${b.notes}</td><td class="text-danger">-₱${val.toFixed(2)}</td></tr>`;
          }
        }
      });

      if (bossAdjustments.filter(b => b.date.startsWith(selectedMonth)).length === 0) {
        bossBody.innerHTML = `<tr><td colspan="3" class="text-center text-muted py-2">Wala pang D/Eco Boss adjustments sa buwang ito.</td></tr>`;
      }

      const grossProfit = totalSales - totalCost;
      const netProfit = grossProfit - totalExpenses + bossNetAdjustment;

      document.getElementById('auditTotalSales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('auditTotalCost').innerText = `₱${totalCost.toFixed(2)}`;
      document.getElementById('auditGrossProfit').innerText = `₱${grossProfit.toFixed(2)}`;
      document.getElementById('auditExpenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${netProfit.toFixed(2)}`;

      const dailyBreakdownBody = document.getElementById('auditDailyBreakdownBody');
      dailyBreakdownBody.innerHTML = '';

      let dailyMap = {};
      transactions.forEach(t => {
        if (t.date.startsWith(selectedMonth)) {
          if (!dailyMap[t.date]) {
            dailyMap[t.date] = { sales: 0, cost: 0, grossProfit: 0 };
          }
          dailyMap[t.date].sales += t.total;
          dailyMap[t.date].cost += (t.totalCost || 0);
          dailyMap[t.date].grossProfit += (t.netProfit || (t.total - (t.totalCost || 0)));
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
        const dayNetProf = item.grossProfit - dayExpSum;

        dailyBreakdownBody.innerHTML += `
          <tr>
            <td class="fw-bold">${d}</td>
            <td class="text-end">₱${item.sales.toFixed(2)}</td>
            <td class="text-end text-secondary">₱${item.cost.toFixed(2)}</td>
            <td class="text-end text-success fw-bold">₱${dayNetProf.toFixed(2)}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-primary py-0 px-2" onclick="jumpToDailyReport('${d}')">
                <i class="fa-solid fa-eye me-1"></i> View
              </button>
            </td>
          </tr>
        `;
      });

      if (sortedDates.length === 0) {
        dailyBreakdownBody.innerHTML = `<tr><td colspan="5" class="text-center text-muted py-3">Walang nahanap na transaksyon sa buwang ito.</td></tr>`;
      }

      renderExpensesTable();
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
      alert('Tagumpay na naidagdag ang D/Eco Boss adjustment!');
    });

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
      document.getElementById('editBalance').value = balance.toFixed(2);
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
        alert('Tagumpay na na-update ang transaksyon!');
      }
    });

    function deleteTransaction(txId) {
      if (confirm('Sigurado ka bang gusto mong tanggalin ang transaksyong ito?')) {
        transactions = transactions.filter(t => t.id !== txId);
        saveData();
        generateDailyReport();
        renderCreditTable();
        alert('Naalis na ang transaksyon.');
      }
    }
  </script>
</body>
</html>
