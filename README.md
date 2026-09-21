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
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTable(); renderStockInHistory();">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item admin-only">
          <button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()">
            <i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit & Net Profit
          </button>
        </li>
      </ul>

      <div class="d-flex align-items-center gap-2">
        <button class="btn btn-success btn-sm fw-semibold" onclick="manualSaveData()" title="Save Data">
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
            <li class="admin-only"><a class="dropdown-item" href="#" onclick="openUserManagementModal()"><i class="fa-solid fa-users-gear me-2"></i>Manage Users</a></li>
            <li><hr class="dropdown-divider"></li>
            <li><a class="dropdown-item text-danger fw-bold" href="#" onclick="logout()"><i class="fa-solid fa-right-from-bracket me-2"></i>Tag Out</a></li>
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
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-cash-register me-2"></i>Record New Transaction / Back-date Entry</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> I-print ang Page
            </button>
          </div>
         
          <div class="alert alert-info py-2 mb-3 d-flex justify-content-between align-items-center">
            <div>
              <i class="fa-solid fa-info-circle me-1"></i> <strong>Paalala:</strong> Maaari kang pumili ng lumang petsa sa ibaba kung nag-e-encode ka ng nakaligtaang araw.
            </div>
            <div class="form-check form-switch mb-0">
              <input class="form-check-input" type="checkbox" id="inventoryOnlyMode" onchange="toggleInventoryOnlyMode()">
              <label class="form-check-label fw-semibold" for="inventoryOnlyMode">Inventory Only Mode</label>
            </div>
          </div>

          <form id="posForm">
            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold text-danger">* Petsa ng Transaksyon (Date of Sale / Log):</label>
                <input type="date" id="saleDate" class="form-control border-danger" required>
                <small class="text-muted">Palitan ito kung nakaligtaang araw ang i-encode mo.</small>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer / Reference Name:</label>
                <input type="text" id="customerName" class="form-control" placeholder="e.g., Juan Dela Cruz" required>
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
                <tbody id="posItemsBody"></tbody>
              </table>
              <button type="button" class="btn btn-sm btn-outline-primary" onclick="addPosRow()">
                <i class="fa-solid fa-plus me-1"></i> Add Another Item
              </button>
            </div>

            <div class="card p-3 bg-light border mb-3" id="containerSectionBox">
              <h6 class="fw-bold text-secondary mb-2"><i class="fa-solid fa-box-open me-2"></i>Container / Lalagyan Details</h6>
              <div class="row g-3">
                <div class="col-md-4">
                  <label class="form-label fw-semibold">Container Status:</label>
                  <select id="containerStatus" class="form-select" onchange="toggleContainerFields()">
                    <option value="NONE">Walang Container / Soli Agad</option>
                    <option value="HIRAM">Hiram / Bagon</option>
                    <option value="DEPOSIT">May Deposito</option>
                  </select>
                </div>
                <div class="col-md-4 container-qty-group" style="display: none;">
                  <label class="form-label fw-semibold">Ilang Container:</label>
                  <input type="number" min="1" id="containerQty" class="form-control" value="1">
                </div>
                <div class="col-md-4 container-deposit-group" style="display: none;">
                  <label class="form-label fw-semibold">Deposito bawat Isa (₱):</label>
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
                  <option value="FULL">Paid in Full (Buong Bayad)</option>
                  <option value="PARTIAL">Partial Payment (May Utang)</option>
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
              <label class="fw-bold me-1">Piliin ang Araw:</label>
              <input type="date" id="dailyReportDate" class="form-control border-primary" onchange="generateDailyReport()">
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

          <!-- MONEY BREAKDOWN -->
          <div class="card p-3 bg-light border mb-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h6 class="fw-bold text-secondary m-0"><i class="fa-solid fa-money-bill-wave me-2"></i>Daily Cash Money Breakdown</h6>
              <button type="button" class="btn btn-sm btn-outline-secondary" onclick="clearMoneyBreakdown()">
                <i class="fa-solid fa-rotate-right me-1"></i> Clear Breakdown
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
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="1000" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱500</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="500" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱200</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="200" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱100</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="100" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱50</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="50" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">₱20</td>
                        <td><input type="number" min="0" class="form-control form-control-sm text-center denom-count" data-denom="20" oninput="calculateMoneyBreakdown()"></td>
                        <td><input type="text" class="form-control form-control-sm bg-light denom-subtotal" readonly value="0.00"></td>
                      </tr>
                      <tr>
                        <td class="fw-semibold text-primary">Coins / Barya</td>
                        <td><span class="text-muted small">Kabuuang Barya</span></td>
                        <td><input type="number" step="0.01" min="0" class="form-control form-control-sm denom-coins" placeholder="0.00" oninput="calculateMoneyBreakdown()"></td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>

              <div class="col-md-5 d-flex flex-column justify-content-between">
                <div class="card p-3 bg-white h-100 border">
                  <h6 class="fw-bold text-dark border-bottom pb-2 mb-3"><i class="fa-solid fa-scale-balanced me-2"></i>Cash Audit & Deductions</h6>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small fw-semibold">Total Collections:</span>
                    <span class="fw-bold text-secondary" id="totalCollectionAll">₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: GCash</span>
                    <span class="text-danger small" id="lessGCash">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="text-muted small">Less: Bank Transfer (BT)</span>
                    <span class="text-danger small" id="lessBT">-₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="text-muted small">Less: Cheque</span>
                    <span class="text-danger small" id="lessCheque">-₱0.00</span>
                  </div>
                  <hr class="my-1">
                  <div class="d-flex justify-content-between align-items-center my-2 bg-light p-2 rounded">
                    <span class="fw-bold text-dark">Target Cash in Drawer:</span>
                    <span class="fs-5 fw-bold text-success" id="breakdownTargetSales">₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="text-muted fw-semibold">Total Cash Counted:</span>
                    <span class="fs-5 fw-bold text-dark" id="totalCountedCash">₱0.00</span>
                  </div>
                  <div class="d-flex justify-content-between align-items-center mb-3">
                    <span class="fw-bold text-dark">Discrepancy:</span>
                    <span class="fs-5 fw-bold" id="cashDiscrepancy">₱0.00</span>
                  </div>
                  <div id="cashStatusAlert" class="alert alert-secondary text-center p-2 fw-bold mb-0">
                    <i class="fa-solid fa-calculator me-1"></i> Magpasok ng breakdown para ma-audit.
                  </div>
                </div>
              </div>
            </div>
          </div>

          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>Listahan ng mga Transaksyon sa Napiling Petsa</h6>
          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle">
              <thead class="table-dark">
                <tr>
                  <th>#</th>
                  <th>Customer Name</th>
                  <th>Location</th>
                  <th>Products Bought</th>
                  <th>Total Cost (₱)</th>
                  <th>Total Amount</th>
                  <th>Paid Amount</th>
                  <th>Balance</th>
                  <th>Net Profit (₱)</th>
                  <th>Payment Method</th>
                  <th class="text-center no-print" style="width: 100px;">Actions</th>
                </tr>
              </thead>
              <tbody id="dailyTableBody"></tbody>
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
              <tbody id="creditTableBody"></tbody>
            </table>
          </div>

          <div class="border-top pt-4">
            <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>Listahan ng mga Nakapagbayad na (Paid Accounts History)</h5>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Petsa ng Hulog / Bayad (Payment Date)</th>
                    <th>Customer Name</th>
                    <th>Product(s)</th>
                    <th>Total Amount (₱)</th>
                    <th>Total Paid (₱)</th>
                    <th>Status</th>
                    <th class="text-center no-print" style="width: 120px;">Actions</th>
                  </tr>
                </thead>
                <tbody id="paidHistoryTableBody"></tbody>
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
                    <p class="mb-1 text-muted small fw-bold">ITEMS BOUGHT:</p>
                    <p class="fs-5 text-dark fw-semibold mb-0" id="lastOrderProducts">-</p>
                  </div>
                </div>
              </div>
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
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory Management</h4>
            <div class="d-flex gap-2">
              <button class="btn btn-outline-secondary" onclick="window.print()"><i class="fa-solid fa-print me-1"></i> Print</button>
              <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#addProductModal"><i class="fa-solid fa-plus me-1"></i> Add Product</button>
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
              <tbody id="inventoryTableBody"></tbody>
              <tfoot class="table-secondary fw-bold text-center" id="inventoryTableFooter"></tfoot>
            </table>
          </div>

          <div class="card p-3 bg-light border mb-4">
            <h5 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-clock-rotate-left me-2"></i>Talaan kung kelan nagdagdag ng Produkto (Stock-In History)</h5>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-light">
                  <tr>
                    <th>Petsa (Date Added)</th>
                    <th>Product Name</th>
                    <th class="text-center">Ibinagdag (Qty)</th>
                    <th>Uri / Note</th>
                  </tr>
                </thead>
                <tbody id="stockInHistoryBody"></tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 5. MONTHLY AUDIT TAB ================= -->
      <div class="tab-pane fade" id="audit-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-chart-pie me-2"></i>Monthly Audit & Net Profit</h4>
            <div class="d-flex gap-2 align-items-center">
              <button class="btn btn-outline-secondary" onclick="window.print()"><i class="fa-solid fa-print me-1"></i> Print</button>
              <label class="fw-bold me-1">Filter Month:</label>
              <input type="month" id="auditMonth" class="form-control" onchange="generateMonthlyAudit()">
            </div>
          </div>

          <div class="row g-3 mb-4">
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">GROSS SALES</span>
                <h5 class="text-primary mt-1 mb-0" id="auditTotalSales">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #6c757d;">
                <span class="text-muted small fw-bold">TOTAL COGS (Puhunan)</span>
                <h5 class="text-secondary mt-1 mb-0" id="auditTotalCost">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;">
                <span class="text-muted small fw-bold">GROSS PROFIT</span>
                <h5 class="text-teal mt-1 mb-0" id="auditGrossProfit" style="color: #00897b;">₱0.00</h5>
              </div>
            </div>
            <div class="col-md-3">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">NET PROFIT</span>
                <h4 class="text-success fw-bold mt-1 mb-0" id="auditNetProfit">₱0.00</h4>
              </div>
            </div>
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
          <h5 class="modal-title"><i class="fa-solid fa-pen-to-square me-2"></i>Edit Transaction & Date</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="editTransactionForm">
          <div class="modal-body">
            <input type="hidden" id="editTxId">
            <div class="mb-3">
              <label class="form-label fw-semibold">Petsa ng Transaksyon (Date):</label>
              <input type="date" id="editDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Customer Name:</label>
              <input type="text" id="editCustomerName" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Products Bought:</label>
              <input type="text" id="editProduct" class="form-control" required>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Total Amount (₱):</label>
                <input type="number" step="0.01" id="editTotal" class="form-control" required oninput="calculateEditBalance()">
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Paid Amount (₱):</label>
                <input type="number" step="0.01" id="editPaid" class="form-control" required oninput="calculateEditBalance()">
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success">Update</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: ADD PRODUCT -->
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
              <label class="form-label fw-semibold">Petsa ng Pagdaragdag (Date Added):</label>
              <input type="date" id="newProdDate" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Product Name:</label>
              <input type="text" id="newProdName" class="form-control" required>
            </div>
            <div class="row g-2 mb-3">
              <div class="col-md-6">
                <label class="form-label fw-semibold">Cost / Unit (₱):</label>
                <input type="number" step="0.01" id="newProdCost" class="form-control" value="0.00" required>
              </div>
              <div class="col-md-6">
                <label class="form-label fw-semibold">Price / Unit (₱):</label>
                <input type="number" step="0.01" id="newProdPrice" class="form-control" value="0.00" required>
              </div>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">Stock Qty:</label>
              <input type="number" id="newProdStock" class="form-control" value="1" min="1" required>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success">Save Product</button>
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
          <h5 class="modal-title"><i class="fa-solid fa-key me-2"></i>Change Name / Password</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <form id="changeProfileForm" onsubmit="updateProfile(event)">
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label fw-semibold">Name:</label>
              <input type="text" id="profileName" class="form-control" required>
            </div>
            <div class="mb-3">
              <label class="form-label fw-semibold">New Password (iwanang blangko kung hindi papalitan):</label>
              <input type="password" id="profilePassword" class="form-control" placeholder="••••••••">
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-success">Save Changes</button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- MODAL: MANAGE USERS -->
  <div class="modal fade" id="manageUsersModal" tabindex="-1">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <div class="modal-header bg-primary text-white">
          <h5 class="modal-title"><i class="fa-solid fa-users-gear me-2"></i>Manage Users</h5>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <div class="table-responsive mb-4">
            <table class="table table-bordered align-middle">
              <thead class="table-light">
                <tr>
                  <th>Name</th>
                  <th>Username</th>
                  <th>Role</th>
                  <th class="text-center">Action</th>
                </tr>
              </thead>
              <tbody id="userListTableBody"></tbody>
            </table>
          </div>
          <hr>
          <h6 class="fw-bold text-secondary mb-3">Add New User</h6>
          <form id="addUserForm" onsubmit="addNewUser(event)">
            <div class="row g-2">
              <div class="col-md-3">
                <input type="text" id="newUserNameInput" class="form-control" placeholder="Full Name" required>
              </div>
              <div class="col-md-3">
                <input type="text" id="newUserUsernameInput" class="form-control" placeholder="Username" required>
              </div>
              <div class="col-md-3">
                <input type="password" id="newUserPasswordInput" class="form-control" placeholder="Password" required>
              </div>
              <div class="col-md-3">
                <button type="submit" class="btn btn-success w-100">Add User</button>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    let users = JSON.parse(localStorage.getItem('rmv_users')) || [
      { id: 1, name: "System Administrator", username: "admin", password: "password", role: "Admin" }
    ];
    let currentUser = JSON.parse(localStorage.getItem('rmv_current_user')) || null;

    let transactions = JSON.parse(localStorage.getItem('rmv_transactions')) || [];
    let inventory = JSON.parse(localStorage.getItem('rmv_inventory')) || [];
    let stockInHistory = JSON.parse(localStorage.getItem('rmv_stockInHistory')) || [];
    let cashBreakdownData = JSON.parse(localStorage.getItem('rmv_cashBreakdownData')) || {};

    function getTodayDateString() {
      const now = new Date();
      return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`;
    }

    let todayFormatted = getTodayDateString();
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;
    document.getElementById('newProdDate').value = todayFormatted;

    window.onload = function() {
      addPosRow();
      checkAuthStatus();
      generateDailyReport();
      renderCreditTable();
      renderInventoryTable();
      renderStockInHistory();
    };

    function checkAuthStatus() {
      const overlay = document.getElementById('loginOverlay');
      if (!currentUser) {
        overlay.style.display = 'flex';
      } else {
        overlay.style.display = 'none';
        document.getElementById('currentUserName').innerText = currentUser.name;
        applyRolePermissions();
      }
    }

    function applyRolePermissions() {
      const adminElements = document.querySelectorAll('.admin-only');
      adminElements.forEach(el => {
        if (currentUser && currentUser.role === 'Admin') {
          el.style.display = '';
        } else {
          el.style.display = 'none';
        }
      });
    }

    document.getElementById('loginForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const u = document.getElementById('loginUsername').value.trim();
      const p = document.getElementById('loginPassword').value.trim();
      const errDiv = document.getElementById('loginError');

      const found = users.find(x => x.username === u && x.password === p);
      if (found) {
        currentUser = found;
        localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
        errDiv.classList.add('d-none');
        this.reset();
        checkAuthStatus();
      } else {
        errDiv.classList.remove('d-none');
      }
    });

    function logout() {
      currentUser = null;
      localStorage.removeItem('rmv_current_user');
      document.getElementById('loginOverlay').style.display = 'flex';
    }

    function openChangeProfileModal() {
      if(!currentUser) return;
      document.getElementById('profileName').value = currentUser.name;
      document.getElementById('profilePassword').value = '';
      new bootstrap.Modal(document.getElementById('changeProfileModal')).show();
    }

    function updateProfile(e) {
      e.preventDefault();
      const newName = document.getElementById('profileName').value.trim();
      const newPass = document.getElementById('profilePassword').value.trim();

      currentUser.name = newName;
      if(newPass) currentUser.password = newPass;

      // Update in users array
      const idx = users.findIndex(x => x.username === currentUser.username);
      if(idx !== -1) {
        users[idx] = currentUser;
      }

      localStorage.setItem('rmv_users', JSON.stringify(users));
      localStorage.setItem('rmv_current_user', JSON.stringify(currentUser));
      document.getElementById('currentUserName').innerText = currentUser.name;

      bootstrap.Modal.getInstance(document.getElementById('changeProfileModal')).hide();
      alert('Tagumpay na na-update ang profile!');
    }

    function openUserManagementModal() {
      renderUserList();
      new bootstrap.Modal(document.getElementById('manageUsersModal')).show();
    }

    function renderUserList() {
      const tbody = document.getElementById('userListTableBody');
      tbody.innerHTML = '';
      users.forEach(u => {
        tbody.innerHTML += `
          <tr>
            <td class="fw-bold">${u.name}</td>
            <td>${u.username}</td>
            <td><span class="badge ${u.role === 'Admin' ? 'bg-danger' : 'bg-secondary'}">${u.role}</span></td>
            <td class="text-center">
              ${u.username !== 'admin' ? `<button class="btn btn-sm btn-outline-danger border-0" onclick="deleteUser('${u.username}')"><i class="fa-solid fa-trash"></i></button>` : '<span class="text-muted small">Protected</span>'}
            </td>
          </tr>
        `;
      });
    }

    function addNewUser(e) {
      e.preventDefault();
      const name = document.getElementById('newUserNameInput').value.trim();
      const username = document.getElementById('newUserUsernameInput').value.trim();
      const password = document.getElementById('newUserPasswordInput').value.trim();

      if(users.some(x => x.username === username)) {
        alert('Mayroon nang gumagamit ng username na ito!');
        return;
      }

      users.push({ id: Date.now(), name, username, password, role: "Staff" });
      localStorage.setItem('rmv_users', JSON.stringify(users));
      document.getElementById('addUserForm').reset();
      renderUserList();
      alert('Tagumpay na naidagdag ang bagong user!');
    }

    function deleteUser(username) {
      if(confirm(`Sigurado ka bang gusto mong tanggalin si ${username}?`)) {
        users = users.filter(x => x.username !== username);
        localStorage.setItem('rmv_users', JSON.stringify(users));
        renderUserList();
      }
    }

    function saveData() {
      localStorage.setItem('rmv_transactions', JSON.stringify(transactions));
      localStorage.setItem('rmv_inventory', JSON.stringify(inventory));
      localStorage.setItem('rmv_stockInHistory', JSON.stringify(stockInHistory));
      localStorage.setItem('rmv_cashBreakdownData', JSON.stringify(cashBreakdownData));
    }

    function manualSaveData() {
      saveData();
      alert('Tagumpay na nai-save ang data!');
    }

    function addPosRow() {
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now() + Math.random().toString(36).substring(2, 5);
      tbody.insertAdjacentHTML('beforeend', `
        <tr id="row-${rowId}">
          <td><input type="text" class="form-control form-control-sm pos-name" placeholder="Pangalan ng Produkto" required></td>
          <td><input type="text" class="form-control form-control-sm pos-desc" placeholder="Description"></td>
          <td><input type="number" class="form-control form-control-sm pos-qty" value="1" min="1" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-price" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm bg-light pos-subtotal" readonly></td>
          <td class="col-action"><button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="document.getElementById('row-${rowId}').remove(); calculateTotal();"><i class="fa-solid fa-trash-can"></i></button></td>
        </tr>
      `);
      calculateTotal();
    }

    function calculateTotal() {
      let grandTotal = 0;
      document.querySelectorAll('#posItemsBody tr').forEach(row => {
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const price = parseFloat(row.querySelector('.pos-price').value) || 0;
        const sub = qty * price;
        row.querySelector('.pos-subtotal').value = sub.toFixed(2);
        grandTotal += sub;
      });
      document.getElementById('totalAmount').value = grandTotal.toFixed(2);
      calculateBalance();
    }

    function toggleCreditFields() {
      const type = document.getElementById('paymentType').value;
      document.getElementById('creditFieldsSection').style.display = type === 'FULL' ? 'none' : 'block';
      if(type === 'FULL') document.getElementById('amountPaidNow').value = document.getElementById('totalAmount').value;
      calculateBalance();
    }

    function calculateBalance() {
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const type = document.getElementById('paymentType').value;
      let paid = type === 'FULL' ? total : (type === 'CREDIT' ? 0 : (parseFloat(document.getElementById('amountPaidNow').value) || 0));
      document.getElementById('remainingBalance').value = Math.max(0, total - paid).toFixed(2);
    }

    document.getElementById('posForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const saleDate = document.getElementById('saleDate').value;
      const custName = document.getElementById('customerName').value;
      const location = document.getElementById('transactionLocation').value;
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paymentType = document.getElementById('paymentType').value;
      const method = document.getElementById('paymentMethod').value;
      let paid = paymentType === 'FULL' ? total : (paymentType === 'CREDIT' ? 0 : (parseFloat(document.getElementById('amountPaidNow').value) || 0));
      const balance = total - paid;
     
      let productSummary = [];
      let totalCostOfGoods = 0;
      document.querySelectorAll('#posItemsBody tr').forEach(row => {
        const name = row.querySelector('.pos-name').value.trim();
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = parseFloat(row.querySelector('.pos-cost').value) || 0;
        totalCostOfGoods += (qty * cost);
        productSummary.push(`${name} (x${qty})`);
      });

      transactions.push({
        id: Date.now(),
        date: saleDate,
        customer: custName,
        location: location,
        product: productSummary.join(', '),
        total: total,
        totalCost: totalCostOfGoods,
        paid: paid,
        balance: balance,
        status: balance === 0 ? "PAID" : (paid > 0 ? "PARTIAL" : "UNPAID"),
        payments: paid > 0 ? [{ amount: paid, method: method, date: saleDate }] : []
      });

      saveData();
      alert('Tagumpay na na-encode para sa petsang: ' + saleDate);
      this.reset();
      document.getElementById('posItemsBody').innerHTML = '';
      addPosRow();
      document.getElementById('saleDate').value = getTodayDateString();
    });

    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';
      let daySales = 0, dayCollected = 0, dayNet = 0, count = 0;

      transactions.forEach((t, index) => {
        if(t.date === selectedDate) {
          daySales += t.total;
          dayNet += (t.total - (t.totalCost || 0));
          count++;
        }
        t.payments.forEach(p => {
          if(p.date === selectedDate) dayCollected += p.amount;
        });

        if(t.date === selectedDate) {
          tbody.innerHTML += `
            <tr>
              <td>${index + 1}</td>
              <td class="fw-bold">${t.customer}</td>
              <td>${t.location}</td>
              <td>${t.product}</td>
              <td>₱${(t.totalCost || 0).toFixed(2)}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger">₱${t.balance.toFixed(2)}</td>
              <td class="text-success fw-bold">₱${(t.total - (t.totalCost || 0)).toFixed(2)}</td>
              <td>${t.payments.length ? t.payments[t.payments.length-1].method : 'N/A'}</td>
              <td class="text-center no-print">
                <button class="btn btn-sm btn-outline-primary border-0" onclick="openEditModal(${t.id})"><i class="fa-solid fa-pen"></i></button>
              </td>
            </tr>
          `;
        }
      });

      if(!count) tbody.innerHTML = `<tr><td colspan="11" class="text-center text-muted py-3">Walang transaksyon sa petsang ito (${selectedDate}).</td></tr>`;

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTotalNetProfit').innerText = `₱${dayNet.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;
    }

    function renderCreditTable() {
      const creditTbody = document.getElementById('creditTableBody');
      const paidTbody = document.getElementById('paidHistoryTableBody');
      creditTbody.innerHTML = '';
      paidTbody.innerHTML = '';

      transactions.forEach(t => {
        if(t.balance > 0) {
          creditTbody.innerHTML += `
            <tr>
              <td class="fw-bold">${t.customer}</td>
              <td>${t.location}</td>
              <td>${t.product}</td>
              <td>₱${(t.totalCost || 0).toFixed(2)}</td>
              <td class="text-success">₱${t.paid.toFixed(2)}</td>
              <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
              <td>${t.dueDate || 'N/A'}</td>
              <td><span class="badge bg-warning text-dark">May Utang</span></td>
              <td><button class="btn btn-sm btn-success" onclick="makePayment(${t.id})">Magbayad</button></td>
            </tr>
          `;
        }
       
        t.payments.forEach(p => {
          paidTbody.innerHTML += `
            <tr>
              <td><span class="badge bg-primary">${p.date}</span></td>
              <td class="fw-bold">${t.customer}</td>
              <td>${t.product}</td>
              <td>₱${t.total.toFixed(2)}</td>
              <td class="text-success fw-bold">₱${p.amount.toFixed(2)} (${p.method})</td>
              <td><span class="badge bg-success">Paid / Hulog</span></td>
              <td class="text-center">-</td>
            </tr>
          `;
        });
      });
    }

    function makePayment(id) {
      const t = transactions.find(item => item.id === id);
      if(!t) return;
      const amt = prompt(`Magkano ang ibinayad ni ${t.customer}? (Balance: ₱${t.balance})`, t.balance);
      const payDate = prompt(`Ilagay ang petsa ng pagbabayad (YYYY-MM-DD):`, getTodayDateString());
     
      if(amt && payDate) {
        const payAmt = parseFloat(amt);
        if(payAmt > 0) {
          t.paid += payAmt;
          t.balance = Math.max(0, t.balance - payAmt);
          t.status = t.balance === 0 ? 'PAID' : 'PARTIAL';
          t.payments.push({ amount: payAmt, method: 'Cash', date: payDate });
          saveData();
          renderCreditTable();
          alert('Na-update na ang bayad para sa petsang ' + payDate);
        }
      }
    }

    function renderInventoryTable() {
      const tbody = document.getElementById('inventoryTableBody');
      tbody.innerHTML = '';
      inventory.forEach((item, index) => {
        tbody.innerHTML += `
          <tr>
            <td>${item.name}</td>
            <td>₱${item.cost.toFixed(2)}</td>
            <td>₱${item.price.toFixed(2)}</td>
            <td class="text-center">${item.beginning}</td>
            <td class="text-center text-success">+${item.stockIn}</td>
            <td class="text-center">0</td>
            <td class="text-center fw-bold">${item.ending}</td>
            <td></td>
          </tr>
        `;
      });
    }

    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const pDate = document.getElementById('newProdDate').value;
      const name = document.getElementById('newProdName').value;
      const cost = parseFloat(document.getElementById('newProdCost').value) || 0;
      const price = parseFloat(document.getElementById('newProdPrice').value) || 0;
      const qty = parseInt(document.getElementById('newProdStock').value) || 0;

      inventory.push({ name, cost, price, beginning: qty, stockIn: 0, ending: qty });
      stockInHistory.push({ date: pDate, product: name, qty: qty, note: 'Bagong Produkto' });
      saveData();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      this.reset();
      renderInventoryTable();
    });

    function renderStockInHistory() {
      const tbody = document.getElementById('stockInHistoryBody');
      tbody.innerHTML = '';
      stockInHistory.forEach(s => {
        tbody.innerHTML += `<tr><td>${s.date}</td><td class="fw-bold">${s.product}</td><td class="text-center text-success">+${s.qty}</td><td>${s.note}</td></tr>`;
      });
    }

    function openEditModal(id) {
      const t = transactions.find(x => x.id === id);
      if(!t) return;
      document.getElementById('editTxId').value = t.id;
      document.getElementById('editDate').value = t.date;
      document.getElementById('editCustomerName').value = t.customer;
      document.getElementById('editProduct').value = t.product;
      document.getElementById('editTotal').value = t.total;
      document.getElementById('editPaid').value = t.paid;
      new bootstrap.Modal(document.getElementById('editTransactionModal')).show();
    }

    document.getElementById('editTransactionForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const id = parseInt(document.getElementById('editTxId').value);
      const t = transactions.find(x => x.id === id);
      if(t) {
        t.date = document.getElementById('editDate').value;
        t.customer = document.getElementById('editCustomerName').value;
        t.product = document.getElementById('editProduct').value;
        t.total = parseFloat(document.getElementById('editTotal').value) || 0;
        t.paid = parseFloat(document.getElementById('editPaid').value) || 0;
        t.balance = Math.max(0, t.total - t.paid);
        saveData();
        bootstrap.Modal.getInstance(document.getElementById('editTransactionModal')).hide();
        generateDailyReport();
        alert('Na-update na ang transaksyon.');
      }
    });

    function generateMonthlyAudit() {
      const mVal = document.getElementById('auditMonth').value;
      let totalSales = 0, totalCost = 0;
      transactions.forEach(t => {
        if(t.date.startsWith(mVal)) {
          totalSales += t.total;
          totalCost += (t.totalCost || 0);
        }
      });
      document.getElementById('auditTotalSales').innerText = `₱${totalSales.toFixed(2)}`;
      document.getElementById('auditTotalCost').innerText = `₱${totalCost.toFixed(2)}`;
      document.getElementById('auditGrossProfit').innerText = `₱${(totalSales - totalCost).toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${(totalSales - totalCost).toFixed(2)}`;
    }
  </script>
</body>
</html>
