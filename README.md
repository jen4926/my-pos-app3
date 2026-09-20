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
        <h4 class="fw-bold">RMVillasis POS</h4>
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
        <i class="fa-solid fa-store me-2"></i>RMVillasis POS
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
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTable(); renderCustomerSalesLog();">
            <i class="fa-solid fa-boxes-stacked me-1"></i> Inventory
          </button>
        </li>
        <li class="nav-item admin-only">
          <button class="nav-link" id="audit-tab" data-bs-toggle="pill" data-bs-target="#audit-content" type="button" onclick="generateMonthlyAudit()">
            <i class="fa-solid fa-chart-pie me-1"></i> Monthly Audit & Net Profit
          </button>
        </li>
      </ul>

      <!-- User Profile & Account Controls -->
      <div class="dropdown text-end text-white">
        <a href="#" class="d-block link-light text-decoration-none dropdown-toggle fw-bold" id="userDropdown" data-bs-toggle="dropdown">
          <i class="fa-solid fa-circle-user fa-lg me-1"></i> <span id="currentUserName">User</span>
        </a>
        <ul class="dropdown-menu dropdown-menu-end text-small shadow">
          <li><a class="dropdown-item" href="#" onclick="openChangeProfileModal()"><i class="fa-solid fa-key me-2"></i>Change Name / Password</a></li>
          <li class="admin-only"><a class="dropdown-item" href="#" onclick="openUserManagementModal()"><i class="fa-solid fa-users-gear me-2"></i>Manage Users & Admins</a></li>
          <li><hr class="dropdown-divider"></li>
          <li><a class="dropdown-item text-danger fw-bold" href="#" onclick="logout()"><i class="fa-solid fa-right-from-bracket me-2"></i>Log Out</a></li>
        </ul>
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
          <form id="posForm">
            <div class="row g-3 mb-3">
              <div class="col-md-4">
                <label class="form-label fw-semibold">Date of Sale:</label>
                <input type="date" id="saleDate" class="form-control" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Customer Name:</label>
                <input type="text" id="customerName" class="form-control" placeholder="e.g., Juan Dela Cruz" required>
              </div>
              <div class="col-md-4">
                <label class="form-label fw-semibold">Transaction Location / Uri:</label>
                <select id="transactionLocation" class="form-select" required>
                  <option value="Hiway">Hiway</option>
                  <option value="Byahe">Byahe</option>
                </select>
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

            <!-- CONTAINER / DEPOSIT DETAILS -->
            <div class="card p-3 bg-light border mb-3">
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

            <div class="row g-3 mt-2">
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
            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-money-bill-wave me-2"></i>Daily Cash Money Breakdown (Denominations)</h6>
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

          <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list-check me-2"></i>List of Encoded Transactions Today <small class="text-muted fs-6">(Maaaring i-edit o i-delete ang Detalye at Puhunan)</small></h6>
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
          <div class="table-responsive">
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

          <!-- Result Display Area -->
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

          <!-- MGA BUMILI NG PRODUKTO (CUSTOMER SALES LOG) -->
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
                </tr>
              </thead>
              <tbody id="customerSalesLogBody">
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

  <!-- MODAL: CHANGE PROFILE (Name & Password) -->
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

  <!-- MODAL: USER MANAGEMENT (Admin Only) -->
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
            <input type="text" id="payCustomerName" class="form-control bg-light" readonly required>
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

  <!-- Datalist para sa manual typing ng product suggestions -->
  <datalist id="inventoryProductList">
    <!-- Dynamic options -->
  </datalist>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    // System Users Database
    let users = [
      { id: 1, name: "System Administrator", username: "admin", password: "password", role: "Admin" },
      { id: 2, name: "Juan Cashier", username: "cashier", password: "password", role: "Staff" }
    ];
    let currentUser = null;

    // Empty Initial Databases
    let transactions = [];
    let inventory = [];
    let bossAdjustments = [];
    let monthlyExpensesData = {};

    // Initial Date Configurations
    const today = new Date();
    const todayFormatted = today.toISOString().split('T')[0];
    document.getElementById('saleDate').value = todayFormatted;
    document.getElementById('dailyReportDate').value = todayFormatted;
    document.getElementById('bossDate').value = todayFormatted;
    document.getElementById('auditMonth').value = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}`;

    window.onload = function() {
      updateProductDatalist();
      addPosRow();
    };

    // Update Datalist options for manual input suggestion
    function updateProductDatalist() {
      const datalist = document.getElementById('inventoryProductList');
      if(datalist) {
        datalist.innerHTML = inventory.map(i => `<option value="${i.name}">`).join('');
      }
    }

    // ================= ENTER KEY FOCUS NEXT =================
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
      this.reset();
      renderUserList();
    });

    function deleteUser(index) {
      if(confirm('Sigurado ka bang gusto mong alisin ang user na ito?')) {
        users.splice(index, 1);
        renderUserList();
      }
    }

    // ================= POS LOGIC (MANUAL TYPE PRODUCT NAME) =================
    function addPosRow() {
      updateProductDatalist();
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now() + Math.random().toString(36).substring(2, 5);
      const rowHTML = `
        <tr id="row-${rowId}">
          <td>
            <input type="text" class="form-control form-control-sm pos-name" list="inventoryProductList" placeholder="I-type ang pangalan ng produkto..." oninput="onProductInput(this)" required>
          </td>
          <td><input type="number" class="form-control form-control-sm pos-qty" value="1" min="1" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-price" placeholder="0.00" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm bg-light pos-subtotal" placeholder="0.00" readonly></td>
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

    // Auto-fill cost and price kung sakaling ang tinype ay umiiral na sa inventory
    function onProductInput(inputElement) {
      const val = inputElement.value.trim().toLowerCase();
      const matched = inventory.find(i => i.name.toLowerCase() === val);
      const row = inputElement.closest('tr');
      if (matched) {
        row.querySelector('.pos-cost').value = matched.cost;
        row.querySelector('.pos-price').value = matched.price;
      }
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
      const total = parseFloat(document.getElementById('totalAmount').value) || 0;
      const paymentType = document.getElementById('paymentType').value;
      const method = document.getElementById('paymentMethod').value;
      const saleDate = document.getElementById('saleDate').value;
      const custName = document.getElementById('customerName').value;
      const location = document.getElementById('transactionLocation').value;
      let paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
      
      if (paymentType === 'FULL') paid = total;
      if (paymentType === 'CREDIT') paid = 0;

      const balance = total - paid;
      let status = "PAID";
      if (balance > 0 && paid > 0) status = "PARTIAL";
      if (balance > 0 && paid === 0) status = "UNPAID";

      const itemRows = document.querySelectorAll('#posItemsBody tr');
      let productSummary = [];
      let itemsPurchasedList = [];
      let totalCostOfGoods = 0;

      itemRows.forEach(row => {
        const name = row.querySelector('.pos-name').value.trim();
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = parseFloat(row.querySelector('.pos-cost').value) || 0;
        const price = parseFloat(row.querySelector('.pos-price').value) || 0;

        totalCostOfGoods += (qty * cost);
        if(name) {
          productSummary.push(`${name} (x${qty})`);
          itemsPurchasedList.push({ name: name, qty: qty, location: location });

          const invItem = inventory.find(inv => inv.name.toLowerCase() === name.toLowerCase());
          if(invItem) {
            invItem.ending = Math.max(0, invItem.ending - qty);
          } else {
            // Kung bago ang tinype na produkto, automatic itong madadagdag sa inventory
            inventory.push({
              name: name,
              cost: cost,
              price: price,
              beginning: 0,
              stockIn: 0,
              ending: Math.max(0, 0 - qty)
            });
          }
        }
      });

      const cStatus = document.getElementById('containerStatus').value;
      const cQty = document.getElementById('containerQty').value || 0;
      const cRate = parseFloat(document.getElementById('containerDepositRate').value) || 0;
      let cInfo = "Wala";

      if (cStatus === 'HIRAM') {
        cInfo = `Hiram (${cQty} pcs)`;
      } else if (cStatus === 'DEPOSIT') {
        cInfo = `May Deposito (${cQty} pcs - ₱${(cQty * cRate).toFixed(2)})`;
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
        dueDate: document.getElementById('dueDate').value || 'N/A',
        status: status,
        payments: paymentHistory
      });

      alert('Transaction saved successfully! Naka-deduct na rin sa Inventory.');
      this.reset();
      document.getElementById('posItemsBody').innerHTML = '';
      addPosRow();
      document.getElementById('saleDate').value = todayFormatted;
      toggleContainerFields();
      toggleCreditFields();
    });

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
        const locBadge = t.location === 'Hiway' ? '<span class="badge bg-primary">Hiway</span>' : '<span class="badge bg-info text-dark">Byahe</span>';
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

      calculateMoneyBreakdown();
    }

    // --- EDIT & DELETE FUNCTIONS FOR ENCODED TRANSACTIONS ---
    function openEditTransactionModal(id) {
      const tx = transactions.find(t => t.id === id);
      if(!tx) return;

      document.getElementById('editTxId').value = tx.id;
      document.getElementById('editCustomerName').value = tx.customer;
      document.getElementById('editLocation').value = tx.location || 'Hiway';
      document.getElementById('editProduct').value = tx.product;
      document.getElementById('editContainerInfo').value = tx.containerInfo || '';
      document.getElementById('editTotalCost').value = tx.totalCost || 0;
      document.getElementById('editTotal').value = tx.total;
      document.getElementById('editPaid').value = tx.paid;
      document.getElementById('editBalance').value = tx.balance.toFixed(2);

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
      const id = parseInt(document.getElementById('editTxId').value);
      const tx = transactions.find(t => t.id === id);

      if(tx) {
        tx.customer = document.getElementById('editCustomerName').value;
        tx.location = document.getElementById('editLocation').value;
        tx.product = document.getElementById('editProduct').value;
        tx.containerInfo = document.getElementById('editContainerInfo').value;
        tx.totalCost = parseFloat(document.getElementById('editTotalCost').value) || 0;
        tx.total = parseFloat(document.getElementById('editTotal').value) || 0;
        tx.netProfit = tx.total - tx.totalCost;
        tx.paid = parseFloat(document.getElementById('editPaid').value) || 0;
        tx.balance = Math.max(0, tx.total - tx.paid);
        
        if (tx.balance === 0) tx.status = "PAID";
        else if (tx.paid > 0) tx.status = "PARTIAL";
        else tx.status = "UNPAID";

        if(tx.payments.length > 0) {
          tx.payments[0].amount = tx.paid;
        } else if(tx.paid > 0) {
          tx.payments.push({ amount: tx.paid, method: 'Cash', date: tx.date });
        }

        alert('Transaction and Cost successfully updated!');
        bootstrap.Modal.getInstance(document.getElementById('editTransactionModal')).hide();
        generateDailyReport();
      }
    });

    function deleteTransaction(id) {
      if(confirm('Sigurado ka bang gusto mong tanggalin ang transaksyong ito?')) {
        const index = transactions.findIndex(t => t.id === id);
        if(index > -1) {
          transactions.splice(index, 1);
          generateDailyReport();
        }
      }
    }

    function calculateMoneyBreakdown() {
      let totalCashCounted = 0;
      const denomInputs = document.querySelectorAll('.denom-count');

      denomInputs.forEach(input => {
        const denom = parseFloat(input.getAttribute('data-denom')) || 0;
        const count = parseFloat(input.value) || 0;
        const subtotal = denom * count;
        
        const row = input.closest('tr');
        row.querySelector('.denom-subtotal').value = subtotal.toFixed(2);
        
        totalCashCounted += subtotal;
      });

      const coins = parseFloat(document.querySelector('.denom-coins').value) || 0;
      totalCashCounted += coins;

      document.getElementById('totalCountedCash').innerText = `₱${totalCashCounted.toFixed(2)}`;

      const diff = totalCashCounted - currentTargetCashInDrawer;
      const diffElem = document.getElementById('cashDiscrepancy');
      const alertElem = document.getElementById('cashStatusAlert');

      diffElem.innerText = `₱${Math.abs(diff).toFixed(2)}`;

      if (totalCashCounted === 0 && currentTargetCashInDrawer === 0) {
        diffElem.className = "fs-5 fw-bold text-dark";
        alertElem.className = "alert alert-secondary text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-calculator me-1"></i> Walang koleksyon o breakdown.`;
      } else if (Math.abs(diff) < 0.01) {
        diffElem.className = "fs-5 fw-bold text-success";
        alertElem.className = "alert alert-success text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-circle-check me-1"></i> PANTAY! Tugma ang Cash sa Drawer.`;
      } else if (diff > 0) {
        diffElem.className = "fs-5 fw-bold text-primary";
        alertElem.className = "alert alert-info text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-circle-exclamation me-1"></i> OVER: Mas sobra ang pera sa drawer nang ₱${diff.toFixed(2)}.`;
      } else {
        diffElem.className = "fs-5 fw-bold text-danger";
        alertElem.className = "alert alert-danger text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-triangle-exclamation me-1"></i> SHORT: Kulang ang pera sa drawer nang ₱${Math.abs(diff).toFixed(2)}.`;
      }
    }

    // ================= CUSTOMER ORDER LOOKUP =================
    function searchCustomerOrder() {
      const query = document.getElementById('searchCustomerInput').value.trim().toLowerCase();
      const resultContainer = document.getElementById('searchResultContainer');
      const noFound = document.getElementById('noCustomerFound');

      if (!query) {
        alert('Mangyaring maglagay ng pangalan ng customer.');
        return;
      }

      const matchedTxs = transactions.filter(t => t.customer.toLowerCase().includes(query));

      if (matchedTxs.length === 0) {
        resultContainer.style.display = 'none';
        noFound.classList.remove('d-none');
        return;
      }

      noFound.classList.add('d-none');
      resultContainer.style.display = 'block';

      // Sort by latest
      matchedTxs.sort((a, b) => b.id - a.id);
      const last = matchedTxs[0];

      document.getElementById('lastOrderCustomer').innerText = last.customer;
      document.getElementById('lastOrderDate').innerText = last.date;
      document.getElementById('lastOrderContainer').innerText = last.containerInfo || 'Wala';
      document.getElementById('lastOrderProducts').innerText = last.product;
      document.getElementById('lastOrderTotal').innerText = `₱${last.total.toFixed(2)}`;
      document.getElementById('lastOrderPaid').innerText = `₱${last.paid.toFixed(2)}`;
      document.getElementById('lastOrderBalance').innerText = `₱${last.balance.toFixed(2)}`;

      const badge = document.getElementById('lastOrderBadge');
      if (last.status === 'PAID') {
        badge.className = 'badge bg-success fs-6';
        badge.innerText = 'PAID (Fully Paid)';
      } else if (last.status === 'PARTIAL') {
        badge.className = 'badge bg-warning text-dark fs-6';
        badge.innerText = 'PARTIAL (May Utang)';
      } else {
        badge.className = 'badge bg-danger fs-6';
        badge.innerText = 'UNPAID (Purong Utang)';
      }

      const historyBody = document.getElementById('customerHistoryBody');
      historyBody.innerHTML = '';
      matchedTxs.forEach(t => {
        const netProf = t.netProfit || (t.total - (t.totalCost || 0));
        historyBody.innerHTML += `
          <tr>
            <td>${t.date}</td>
            <td><span class="badge ${t.location === 'Hiway' ? 'bg-primary' : 'bg-info text-dark'}">${t.location}</span></td>
            <td>${t.product}</td>
            <td>${t.containerInfo || 'Wala'}</td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger">₱${t.balance.toFixed(2)}</td>
            <td class="text-success fw-bold">₱${netProf.toFixed(2)}</td>
            <td><span class="badge ${t.status === 'PAID' ? 'bg-success' : (t.status === 'PARTIAL' ? 'bg-warning text-dark' : 'bg-danger')}">${t.status}</span></td>
          </tr>
        `;
      });
    }

    // ================= UTANG LEDGER =================
    function renderCreditTable() {
      const tbody = document.getElementById('creditTableBody');
      tbody.innerHTML = '';

      const unpaidTxs = transactions.filter(t => t.balance > 0);

      unpaidTxs.forEach(t => {
        tbody.innerHTML += `
          <tr>
            <td class="fw-bold">${t.customer}</td>
            <td><span class="badge ${t.location === 'Hiway' ? 'bg-primary' : 'bg-info text-dark'}">${t.location}</span></td>
            <td>${t.product}</td>
            <td class="text-secondary">₱${(t.totalCost || 0).toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
            <td>${t.dueDate}</td>
            <td><span class="badge ${t.status === 'PARTIAL' ? 'bg-warning text-dark' : 'bg-danger'}">${t.status}</span></td>
            <td class="no-print">
              <button class="btn btn-sm btn-success py-0 px-2" onclick="openPaymentModal(${t.id})">
                <i class="fa-solid fa-hand-holding-dollar me-1"></i>Magbayad
              </button>
            </td>
          </tr>
        `;
      });

      if (unpaidTxs.length === 0) {
        tbody.innerHTML = `<tr><td colspan="9" class="text-center text-muted py-3">Walang kasalukuyang utang o balanse ang mga customer.</td></tr>`;
      }
    }

    function openPaymentModal(id) {
      const tx = transactions.find(t => t.id === id);
      if(!tx) return;

      document.getElementById('payTransactionId').value = tx.id;
      document.getElementById('payCustomerName').value = tx.customer;
      document.getElementById('payCurrentBalance').value = tx.balance.toFixed(2);
      document.getElementById('payAmountInput').value = '';

      new bootstrap.Modal(document.getElementById('paymentModal')).show();
    }

    function submitUtangPayment() {
      const id = parseInt(document.getElementById('payTransactionId').value);
      const payAmount = parseFloat(document.getElementById('payAmountInput').value) || 0;
      const method = document.getElementById('payMethod').value;
      const tx = transactions.find(t => t.id === id);

      if (!tx) return;
      if (payAmount <= 0) {
        alert('Mangyaring maglagay ng wastong halaga ng kabayaran.');
        return;
      }
      if (payAmount > tx.balance) {
        alert('Ang ibinayad ay mas malaki kaysa sa natitirang balanse.');
        return;
      }

      tx.paid += payAmount;
      tx.balance = Math.max(0, tx.total - tx.paid);
      if (tx.balance === 0) {
        tx.status = 'PAID';
      } else {
        tx.status = 'PARTIAL';
      }

      tx.payments.push({
        amount: payAmount,
        method: method,
        date: todayFormatted
      });

      alert('Tagumpay na nai-record ang kabayaran sa utang!');
      bootstrap.Modal.getInstance(document.getElementById('paymentModal')).hide();
      renderCreditTable();
    }

    // ================= INVENTORY MANAGEMENT =================
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const name = document.getElementById('newProdName').value.trim();
      const cost = parseFloat(document.getElementById('newProdCost').value) || 0;
      const price = parseFloat(document.getElementById('newProdPrice').value) || 0;
      const stock = parseInt(document.getElementById('newProdStock').value) || 0;

      const existing = inventory.find(i => i.name.toLowerCase() === name.toLowerCase());
      if (existing) {
        alert('Ang produktong ito ay umiiral na sa inventory!');
        return;
      }

      inventory.push({
        name: name,
        cost: cost,
        price: price,
        beginning: stock,
        stockIn: 0,
        ending: stock
      });

      alert('Matagumpay na naidagdag ang bagong produkto!');
      this.reset();
      bootstrap.Modal.getInstance(document.getElementById('addProductModal')).hide();
      renderInventoryTable();
      updateProductDatalist();
    });

    function renderInventoryTable() {
      const tbody = document.getElementById('inventoryTableBody');
      const tfoot = document.getElementById('inventoryTableFooter');
      tbody.innerHTML = '';

      let totalBeg = 0;
      let totalStockIn = 0;
      let totalSold = 0;
      let totalEnd = 0;
      let totalInventoryAsset = 0;

      inventory.forEach((item, index) => {
        const soldQty = item.beginning + item.stockIn - item.ending;
        totalBeg += item.beginning;
        totalStockIn += item.stockIn;
        totalSold += Math.max(0, soldQty);
        totalEnd += item.ending;
        totalInventoryAsset += (item.ending * item.cost);

        tbody.innerHTML += `
          <tr>
            <td class="fw-semibold">${item.name}</td>
            <td class="text-center">₱<input type="number" step="0.01" class="form-control form-control-sm d-inline-block inventory-input" value="${item.cost}" onchange="updateInventoryValue(${index}, 'cost', this.value)"></td>
            <td class="text-center">₱<input type="number" step="0.01" class="form-control form-control-sm d-inline-block inventory-input" value="${item.price}" onchange="updateInventoryValue(${index}, 'price', this.value)"></td>
            <td class="text-center">${item.beginning}</td>
            <td class="text-center"><input type="number" min="0" class="form-control form-control-sm d-inline-block inventory-input" value="${item.stockIn}" onchange="updateInventoryValue(${index}, 'stockIn', this.value)"></td>
            <td class="text-center fw-bold text-primary">${Math.max(0, soldQty)}</td>
            <td class="text-center fw-bold text-success">${item.ending}</td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-0" onclick="deleteInventoryItem(${index})"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      if (inventory.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Walang nakatalang produkto sa inventory.</td></tr>`;
      }

      tfoot.innerHTML = `
        <tr>
          <td class="text-start">TOTALS & INVENTORY ASSET:</td>
          <td colspan="2" class="text-start text-primary">Est. Asset Value: ₱${totalInventoryAsset.toFixed(2)}</td>
          <td class="text-center">${totalBeg}</td>
          <td class="text-center">${totalStockIn}</td>
          <td class="text-center">${totalSold}</td>
          <td class="text-center">${totalEnd}</td>
          <td class="no-print"></td>
        </tr>
      `;
    }

    function updateInventoryValue(index, field, value) {
      const val = parseFloat(value) || 0;
      inventory[index][field] = val;
      if (field === 'stockIn') {
        // Recalculate ending stock automatically
        const item = inventory[index];
        // total sold can be derived
        let sold = (item.beginning + item.stockIn) - item.ending;
        if(sold < 0) sold = 0;
      }
      renderInventoryTable();
    }

    function deleteInventoryItem(index) {
      if(confirm('Sigurado ka bang gusto mong tanggalin ang produktong ito sa inventory?')) {
        inventory.splice(index, 1);
        renderInventoryTable();
        updateProductDatalist();
      }
    }

    function renderCustomerSalesLog() {
      const tbody = document.getElementById('customerSalesLogBody');
      tbody.innerHTML = '';

      let hasLogs = false;
      transactions.forEach(t => {
        if (t.itemsList && t.itemsList.length > 0) {
          t.itemsList.forEach(item => {
            hasLogs = true;
            tbody.innerHTML += `
              <tr>
                <td>${t.date}</td>
                <td class="fw-bold">${t.customer}</td>
                <td><span class="badge ${t.location === 'Hiway' ? 'bg-primary' : 'bg-info text-dark'}">${t.location}</span></td>
                <td>${item.name}</td>
                <td class="text-center fw-bold">${item.qty}</td>
              </tr>
            `;
          });
        }
      });

      if (!hasLogs) {
        tbody.innerHTML = `<tr><td colspan="5" class="text-center text-muted py-3">Wala pang nakatalang benta ng produkto sa mga customer.</td></tr>`;
      }
    }

    // ================= MONTHLY AUDIT & NET PROFIT =================
    function addExpenseRow() {
      const monthKey = document.getElementById('auditMonth').value;
      if (!monthlyExpensesData[monthKey]) {
        monthlyExpensesData[monthKey] = [];
      }
      monthlyExpensesData[monthKey].push({ name: '', amount: 0 });
      renderExpenseTable(monthKey);
    }

    function renderExpenseTable(monthKey) {
      const tbody = document.getElementById('expenseTableBody');
      tbody.innerHTML = '';

      if (!monthlyExpensesData[monthKey]) {
        monthlyExpensesData[monthKey] = [];
      }

      const expenses = monthlyExpensesData[monthKey];

      expenses.forEach((exp, index) => {
        tbody.innerHTML += `
          <tr>
            <td><input type="text" class="form-control form-control-sm" value="${exp.name}" placeholder="Hal. Kuryente, Sahod sa Tauhan" onchange="updateExpenseItem(${index}, 'name', this.value)"></td>
            <td><input type="number" step="0.01" class="form-control form-control-sm" value="${exp.amount}" placeholder="0.00" onchange="updateExpenseItem(${index}, 'amount', this.value)"></td>
            <td class="text-center no-print">
              <button class="btn btn-sm btn-outline-danger border-0 p-0" onclick="removeExpenseRow(${index})"><i class="fa-solid fa-trash"></i></button>
            </td>
          </tr>
        `;
      });

      if (expenses.length === 0) {
        tbody.innerHTML = `<tr><td colspan="3" class="text-center text-muted py-2">Walang nakatalang expenses sa buwang ito.</td></tr>`;
      }

      calculateAuditSummary(monthKey);
    }

    function updateExpenseItem(index, field, value) {
      const monthKey = document.getElementById('auditMonth').value;
      if (field === 'amount') {
        monthlyExpensesData[monthKey][index][field] = parseFloat(value) || 0;
      } else {
        monthlyExpensesData[monthKey][index][field] = value;
      }
      calculateAuditSummary(monthKey);
    }

    function removeExpenseRow(index) {
      const monthKey = document.getElementById('auditMonth').value;
      monthlyExpensesData[monthKey].splice(index, 1);
      renderExpenseTable(monthKey);
    }

    // Boss Adjustment Form Handler
    document.getElementById('bossForm').addEventListener('submit', function(e) {
      e.preventDefault();
      bossAdjustments.push({
        date: document.getElementById('bossDate').value,
        type: document.getElementById('bossType').value,
        amount: parseFloat(document.getElementById('bossAmount').value) || 0,
        notes: document.getElementById('bossNotes').value || 'Boss Adjustment'
      });

      alert('Boss adjustment successfully saved!');
      this.reset();
      document.getElementById('bossDate').value = todayFormatted;
      bootstrap.Modal.getInstance(document.getElementById('bossModal')).hide();
      generateMonthlyAudit();
    });

    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('auditMonth').value; // Format: YYYY-MM
      renderExpenseTable(selectedMonth);
    }

    function calculateAuditSummary(monthKey) {
      let grossSales = 0;
      let totalCost = 0;

      transactions.forEach(t => {
        if (t.date && t.date.startsWith(monthKey)) {
          grossSales += t.total;
          totalCost += (t.totalCost || 0);
        }
      });

      const grossProfit = grossSales - totalCost;

      // Calculate Expenses
      let totalExpenses = 0;
      if (monthlyExpensesData[monthKey]) {
        monthlyExpensesData[monthKey].forEach(exp => {
          totalExpenses += (exp.amount || 0);
        });
      }

      // Boss Adjustments for the month
      let bossAdd = 0;
      let bossSub = 0;
      const bossBody = document.getElementById('bossLogsBody');
      bossBody.innerHTML = '';

      bossAdjustments.forEach(b => {
        if (b.date && b.date.startsWith(monthKey)) {
          if (b.type === 'ADD') bossAdd += b.amount;
          else bossSub += b.amount;

          bossBody.innerHTML += `
            <tr>
              <td>${b.date}</td>
              <td><span class="badge ${b.type === 'ADD' ? 'bg-success' : 'bg-danger'}">${b.type === 'ADD' ? 'Capital In (+)' : 'Withdrawal (-)'}</span> ${b.notes}</td>
              <td>₱${b.amount.toFixed(2)}</td>
            </tr>
          `;
        }
      });

      if (bossAdjustments.filter(b => b.date && b.date.startsWith(monthKey)).length === 0) {
        bossBody.innerHTML = `<tr><td colspan="3" class="text-center text-muted py-2">Walang boss adjustment sa buwang ito.</td></tr>`;
      }

      const netProfit = grossProfit - totalExpenses + bossAdd - bossSub;

      document.getElementById('auditTotalSales').innerText = `₱${grossSales.toFixed(2)}`;
      document.getElementById('auditTotalCost').innerText = `₱${totalCost.toFixed(2)}`;
      document.getElementById('auditGrossProfit').innerText = `₱${grossProfit.toFixed(2)}`;
      document.getElementById('auditExpenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${netProfit.toFixed(2)}`;
    }
  </script>
</body>
</html>
