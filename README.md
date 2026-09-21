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
    .inventory-input { width: 85px; text-align: center; }
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
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-cash-register me-2"></i>Record New Transaction</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> I-print ang Page / Resibo
            </button>
          </div>
          
          <!-- Inventory Only Mode Toggle -->
          <div class="alert alert-info py-2 mb-3 d-flex justify-content-between align-items-center">
            <div>
              <i class="fa-solid fa-info-circle me-1"></i> <strong>Mode:</strong> Pwede kang magpasok ng item para sa Inventory Lang (Walang halaga/presyo).
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
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light">
                <span class="text-muted small fw-bold">DAILY TOTAL SALES</span>
                <h4 class="text-primary mt-1 mb-0" id="dailyTotalSales">₱0.00</h4>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                <span class="text-muted small fw-bold">TOTAL COLLECTION (All Payments)</span>
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
                  </table>
                </div>
              </div>

              <!-- CASH VERIFICATION / DISCREPANCY COMPARISON -->
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

          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>List of Encoded Transactions Today</h6>
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
                    <th class="col-action no-print"><i class="fa-solid fa-trash"></i></th>
                  </tr>
                </thead>
                <tbody id="expenseTableBody">
                  <!-- Dynamic Expense Rows -->
                </tbody>
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
    // System Users Database
    let users = JSON.parse(localStorage.getItem('rmv_users')) || [
      { id: 1, name: "System Administrator", username: "admin", password: "password", role: "Admin" },
      { id: 2, name: "Juan Cashier", username: "cashier", password: "password", role: "Staff" }
    ];
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
    const nowObj = new Date();
    document.getElementById('auditMonth').value = `${nowObj.getFullYear()}-${String(nowObj.getMonth() + 1).padStart(2, '0')}`;

    function checkAndUpdateDate() {
      const currentDate = getTodayDateString();
      const saleDateInput = document.getElementById('saleDate');
      
      if (saleDateInput && saleDateInput.value === todayFormatted) {
        saleDateInput.value = currentDate;
      }
      todayFormatted = currentDate;
    }

    setInterval(checkAndUpdateDate, 60000);

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
        const inputs = Array.from(document.querySelectorAll('.denom-count, .denom-coins'));
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

      // Refresh rows display for price columns
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

          // Update Inventory Stock (Add or Deduct based on transaction type)
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

          // Kung Inventory Only, itala rin sa Stock-In History
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

    // ================= ADD PRODUCT & STOCK-IN HISTORY LOGIC =================
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const name = document.getElementById('newProdName').value.trim();
      const cost = parseFloat(document.getElementById('newProdCost').value) || 0;
      const price = parseFloat(document.getElementById('newProdPrice').value) || 0;
      const qty = parseInt(document.getElementById('newProdStock').value) || 0;
      const currentDate = getTodayDateString();

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
          date: currentDate,
          product: name,
          qty: qty,
          note: existing ? 'Nagdagdag ng Stock' : 'Bagong Produkto / Beginning Stock'
        });
      }

      saveData();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      this.reset();
      renderInventoryTable();
      renderStockInHistory();
      alert('Tagumpay na naidagdag ang produkto at nailagay sa talaan!');
    });

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

    // ================= DAILY REPORT & MONEY BREAKDOWN LOGIC =================
    let currentTargetCashInDrawer = 0;

    function generateDailyReport() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const tbody = document.getElementById('dailyTableBody');
      tbody.innerHTML = '';

      let daySales = 0;
      let dayCollected = 0;
      let count = 0;

      let totalGCash = 0;
      let totalBT = 0;
      let totalCheque = 0;

      const filtered = transactions.filter(t => t.date === selectedDate || t.payments.some(p => p.date === selectedDate));

      filtered.forEach((t, index) => {
        if(t.date === selectedDate) daySales += t.total;
        
        t.payments.forEach(p => {
          if (p.date === selectedDate) {
            dayCollected += p.amount;
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

        const txCost = t.totalCost || 0;
        const netProf = t.total - txCost;

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

      currentTargetCashInDrawer = Math.max(0, dayCollected - (totalGCash + totalBT + totalCheque));

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;

      document.getElementById('totalCollectionAll').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('lessGCash').innerText = `-₱${totalGCash.toFixed(2)}`;
      document.getElementById('lessBT').innerText = `-₱${totalBT.toFixed(2)}`;
      document.getElementById('lessCheque').innerText = `-₱${totalCheque.toFixed(2)}`;
      document.getElementById('breakdownTargetSales').innerText = `₱${currentTargetCashInDrawer.toFixed(2)}`;

      loadMoneyBreakdown();
    }

    function calculateMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const countInputs = document.querySelectorAll('.denom-count');
      const subtotalInputs = document.querySelectorAll('.denom-subtotal');
      const coinsInput = document.querySelector('.denom-coins');

      let totalCounted = 0;
      let breakdownState = { counts: {}, coins: 0 };

      countInputs.forEach((input, index) => {
        const denom = parseFloat(input.getAttribute('data-denom')) || 0;
        const count = parseFloat(input.value) || 0;
        const subtotal = denom * count;
        
        subtotalInputs[index].value = subtotal.toFixed(2);
        totalCounted += subtotal;

        breakdownState.counts[denom] = count;
      });

      const coinsValue = parseFloat(coinsInput.value) || 0;
      totalCounted += coinsValue;
      breakdownState.coins = coinsValue;

      // I-save ang state para sa kasalukuyang petsa
      cashBreakdownData[selectedDate] = breakdownState;
      saveData();

      document.getElementById('totalCountedCash').innerText = `₱${totalCounted.toFixed(2)}`;

      const discrepancy = totalCounted - currentTargetCashInDrawer;
      const discEl = document.getElementById('cashDiscrepancy');
      const statusAlert = document.getElementById('cashStatusAlert');

      discEl.innerText = `₱${discrepancy.toFixed(2)}`;

      if (Math.abs(discrepancy) < 0.01) {
        discEl.className = "fs-5 fw-bold text-success";
        statusAlert.className = "alert alert-success text-center p-2 fw-bold mb-0";
        statusAlert.innerHTML = `<i class="fa-solid fa-circle-check me-1"></i> Balanse ang Cash sa Drawer! (Exact Match)`;
      } else if (discrepancy > 0) {
        discEl.className = "fs-5 fw-bold text-primary";
        statusAlert.className = "alert alert-warning text-center p-2 fw-bold mb-0";
        statusAlert.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> May Sobra (Over) na ₱${discrepancy.toFixed(2)}`;
      } else {
        discEl.className = "fs-5 fw-bold text-danger";
        statusAlert.className = "alert alert-danger text-center p-2 fw-bold mb-0";
        statusAlert.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> May Kulang (Short) na ₱${Math.abs(discrepancy).toFixed(2)}`;
      }
    }

    function loadMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      const countInputs = document.querySelectorAll('.denom-count');
      const coinsInput = document.querySelector('.denom-coins');

      const dayData = cashBreakdownData[selectedDate] || { counts: {}, coins: 0 };

      countInputs.forEach(input => {
        const denom = input.getAttribute('data-denom');
        input.value = dayData.counts[denom] !== undefined ? dayData.counts[denom] : '';
      });

      coinsInput.value = dayData.coins ? dayData.coins : '';
      calculateMoneyBreakdown();
    }

    function clearMoneyBreakdown() {
      const selectedDate = document.getElementById('dailyReportDate').value;
      if (confirm('Sigurado ka bang gusto mong i-clear ang cash breakdown para sa petsang ito?')) {
        delete cashBreakdownData[selectedDate];
        saveData();
        loadMoneyBreakdown();
      }
    }
  </script>
</body>
</html>
