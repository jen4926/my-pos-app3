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
    .inventory-input { width: 75px; text-align: center; }
    .stat-card { border-left: 4px solid #1976d2; }
    
    /* Login Backdrop overlay */
    #loginOverlay {
      position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
      background: rgba(13, 71, 161, 0.85); z-index: 9999;
      display: flex; justify-content: center; align-items: center;
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
          <button class="nav-link" id="inventory-tab" data-bs-toggle="pill" data-bs-target="#inventory-content" type="button" onclick="renderInventoryTable()">
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

              <!-- CASH VERIFICATION / DISCREPANCY COMPARISON -->
              <div class="col-md-5 d-flex flex-column justify-content-between">
                <div class="card p-3 bg-white h-100 border">
                  <h6 class="fw-bold text-dark border-bottom pb-2 mb-3"><i class="fa-solid fa-scale-balanced me-2"></i>Cash Audit & Comparison</h6>
                  
                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="text-muted fw-semibold">Total Cash Counted:</span>
                    <span class="fs-5 fw-bold text-dark" id="totalCountedCash">₱0.00</span>
                  </div>

                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="text-muted fw-semibold">Daily Cash Collected:</span>
                    <span class="fs-5 fw-bold text-success" id="breakdownTargetSales">₱0.00</span>
                  </div>

                  <hr class="my-2">

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
                  <th>Products Bought</th>
                  <th>Container Status</th>
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

      <!-- ================= 3.5 CUSTOMER ORDER LOOKUP TAB ================= -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <h4 class="card-title text-primary mb-4"><i class="fa-solid fa-magnifying-glass me-2"></i>Track Customer Last Order</h4>
          
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

            <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-clock-rotate-left me-2"></i>Lahat ng Naging Transaksyon ni Customer</h6>
            <div class="table-responsive">
              <table class="table table-bordered table-hover align-middle bg-white">
                <thead class="table-dark">
                  <tr>
                    <th>Date</th>
                    <th>Products</th>
                    <th>Container</th>
                    <th>Total (₱)</th>
                    <th>Paid (₱)</th>
                    <th>Balance (₱)</th>
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
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-chart-pie me-2"></i>Monthly Audit & Net Profit Computation</h4>
            <div class="d-flex gap-2 align-items-center">
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
                    <th class="col-action"><i class="fa-solid fa-trash"></i></th>
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
            <input type="text" id="payCustomerName" class="form-control" placeholder="e.g., Juan Dela Cruz" required>
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
        
        // Hide Admin controls if staff
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

    // ================= POS LOGIC =================
    function addPosRow() {
      const tbody = document.getElementById('posItemsBody');
      const rowId = Date.now();
      const rowHTML = `
        <tr id="row-${rowId}">
          <td><input type="text" class="form-control form-control-sm pos-name" placeholder="Product Name" required></td>
          <td><input type="number" class="form-control form-control-sm pos-qty" value="1" min="1" oninput="calculateTotal()" required></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm pos-cost" placeholder="0.00" oninput="calculateTotal()" required></td>
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
      let paid = parseFloat(document.getElementById('amountPaidNow').value) || 0;
      
      if (paymentType === 'FULL') paid = total;
      if (paymentType === 'CREDIT') paid = 0;

      const balance = total - paid;
      let status = "PAID";
      if (balance > 0 && paid > 0) status = "PARTIAL";
      if (balance > 0 && paid === 0) status = "UNPAID";

      const itemRows = document.querySelectorAll('#posItemsBody tr');
      let productSummary = [];
      let totalCostOfGoods = 0;

      itemRows.forEach(row => {
        const name = row.querySelector('.pos-name').value;
        const qty = parseFloat(row.querySelector('.pos-qty').value) || 0;
        const cost = parseFloat(row.querySelector('.pos-cost').value) || 0;

        totalCostOfGoods += (qty * cost);
        if(name) productSummary.push(`${name} (x${qty})`);
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
        id: transactions.length + 1,
        date: saleDate,
        customer: document.getElementById('customerName').value,
        product: productSummary.join(', '),
        containerInfo: cInfo,
        total: total,
        totalCost: totalCostOfGoods,
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
      toggleContainerFields();
      toggleCreditFields();
    });

    // ================= DAILY REPORT & MONEY BREAKDOWN LOGIC =================
    let currentDailyCollected = 0;

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
            <td><span class="badge bg-secondary">${t.containerInfo || 'Wala'}</span></td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger">₱${t.balance.toFixed(2)}</td>
            <td>${t.payments.length > 0 ? t.payments[0].method : 'N/A'}</td>
          </tr>
        `;
      });

      if (filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="8" class="text-center text-muted py-3">Walang na-encode na transaksyon sa petsang ito.</td></tr>`;
      }

      currentDailyCollected = dayCollected;

      document.getElementById('dailyTotalSales').innerText = `₱${daySales.toFixed(2)}`;
      document.getElementById('dailyTotalCollected').innerText = `₱${dayCollected.toFixed(2)}`;
      document.getElementById('dailyTxCount').innerText = count;
      document.getElementById('breakdownTargetSales').innerText = `₱${dayCollected.toFixed(2)}`;

      calculateMoneyBreakdown();
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

      const diff = totalCashCounted - currentDailyCollected;
      const diffElem = document.getElementById('cashDiscrepancy');
      const alertElem = document.getElementById('cashStatusAlert');

      diffElem.innerText = `₱${Math.abs(diff).toFixed(2)}`;

      if (totalCashCounted === 0 && currentDailyCollected === 0) {
        diffElem.className = "fs-5 fw-bold text-dark";
        alertElem.className = "alert alert-secondary text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-calculator me-1"></i> Walang koleksyon o breakdown.`;
      } else if (Math.abs(diff) < 0.01) {
        diffElem.className = "fs-5 fw-bold text-success";
        alertElem.className = "alert alert-success text-center p-2 fw-bold mb-0";
        alertElem.innerHTML = `<i class="fa-solid fa-circle-check me-1"></i> PANTAY! Tugma ang pera sa Daily Collection.`;
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
      const rowId = Date.now() + Math.random().toString(36).substring(2, 7);
      const rowHTML = `
        <tr id="exp-${rowId}">
          <td><input type="text" class="form-control form-control-sm exp-name" placeholder="e.g., Sahod ni Juan, Kuryente, etc." value="${name}" onchange="saveCurrentMonthExpenses()"></td>
          <td><input type="number" step="0.01" class="form-control form-control-sm exp-amount" placeholder="0.00" value="${amount}" onchange="saveCurrentMonthExpenses()" oninput="updateCalculationsOnly()"></td>
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
      updateCalculationsOnly();
    }

    function updateCalculationsOnly() {
      const selectedMonth = document.getElementById('auditMonth').value;
      if (!selectedMonth) return;

      let grossSales = 0;
      let totalCost = 0;

      transactions.forEach(tx => {
        if (tx.date.startsWith(selectedMonth)) {
          grossSales += tx.total;
          totalCost += (tx.totalCost || 0);
        }
      });

      let grossProfit = grossSales - totalCost;

      let totalExpenses = 0;
      const rows = document.querySelectorAll('#expenseTableBody tr');
      rows.forEach(row => {
        totalExpenses += parseFloat(row.querySelector('.exp-amount').value) || 0;
      });

      let bossTotal = 0;
      bossAdjustments.filter(b => b.date.startsWith(selectedMonth)).forEach(b => {
        const val = b.type === 'ADD' ? b.amount : -b.amount;
        bossTotal += val;
      });

      const netProfit = grossProfit - totalExpenses + bossTotal;

      document.getElementById('auditTotalSales').innerText = `₱${grossSales.toFixed(2)}`;
      document.getElementById('auditTotalCost').innerText = `₱${totalCost.toFixed(2)}`;
      document.getElementById('auditGrossProfit').innerText = `₱${grossProfit.toFixed(2)}`;
      document.getElementById('auditExpenses').innerText = `₱${totalExpenses.toFixed(2)}`;
      document.getElementById('auditNetProfit').innerText = `₱${netProfit.toFixed(2)}`;
    }

    // ================= MONTHLY AUDIT & NET PROFIT =================
    function generateMonthlyAudit() {
      const selectedMonth = document.getElementById('auditMonth').value;
      if (!selectedMonth) return;

      const expBody = document.getElementById('expenseTableBody');
      expBody.innerHTML = '';
      const monthExp = monthlyExpensesData[selectedMonth] || [];

      if (monthExp.length === 0) {
        addExpenseRow();
      } else {
        monthExp.forEach(exp => {
          const rowId = Date.now() + Math.random().toString(36).substring(2, 7);
          expBody.innerHTML += `
            <tr id="exp-${rowId}">
              <td><input type="text" class="form-control form-control-sm exp-name" placeholder="Expense Name" value="${exp.name}" onchange="saveCurrentMonthExpenses()"></td>
              <td><input type="number" step="0.01" class="form-control form-control-sm exp-amount" placeholder="0.00" value="${exp.amount || ''}" onchange="saveCurrentMonthExpenses()" oninput="updateCalculationsOnly()"></td>
              <td class="col-action">
                <button type="button" class="btn btn-sm btn-outline-danger border-0 p-1" onclick="removeExpenseRow('exp-${rowId}')">
                  <i class="fa-solid fa-trash-can"></i>
                </button>
              </td>
            </tr>
          `;
        });
      }

      const bossBody = document.getElementById('bossLogsBody');
      bossBody.innerHTML = '';

      bossAdjustments.filter(b => b.date.startsWith(selectedMonth)).forEach(b => {
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

      updateCalculationsOnly();
    }

    // ================= SEARCH & TRACK CUSTOMER ORDER =================
    function searchCustomerOrder() {
      const query = document.getElementById('searchCustomerInput').value.trim().toLowerCase();
      const resultContainer = document.getElementById('searchResultContainer');
      const noResultAlert = document.getElementById('noCustomerFound');
      const historyBody = document.getElementById('customerHistoryBody');

      if (!query) {
        alert('Mangyaring maglagay ng pangalan ng customer.');
        return;
      }

      // Filter transactions matching customer name
      const customerTxs = transactions.filter(t => t.customer.toLowerCase().includes(query));

      if (customerTxs.length === 0) {
        resultContainer.style.display = 'none';
        noResultAlert.classList.remove('d-none');
        return;
      }

      noResultAlert.classList.add('d-none');
      resultContainer.style.display = 'block';

      // Sort by latest transaction first
      customerTxs.sort((a, b) => b.id - a.id);

      const lastOrder = customerTxs[0];

      // Populate Last Order Card
      document.getElementById('lastOrderCustomer').innerText = lastOrder.customer;
      document.getElementById('lastOrderDate').innerText = lastOrder.date;
      document.getElementById('lastOrderContainer').innerText = lastOrder.containerInfo || 'Wala';
      document.getElementById('lastOrderProducts').innerText = lastOrder.product;
      document.getElementById('lastOrderTotal').innerText = `₱${lastOrder.total.toFixed(2)}`;
      document.getElementById('lastOrderPaid').innerText = `₱${lastOrder.paid.toFixed(2)}`;
      document.getElementById('lastOrderBalance').innerText = `₱${lastOrder.balance.toFixed(2)}`;

      const badgeElem = document.getElementById('lastOrderBadge');
      badgeElem.innerText = lastOrder.status;
      if (lastOrder.status === 'PAID') {
        badgeElem.className = 'badge bg-success fs-6';
      } else if (lastOrder.status === 'PARTIAL') {
        badgeElem.className = 'badge bg-warning text-dark fs-6';
      } else {
        badgeElem.className = 'badge bg-danger fs-6';
      }

      // Populate History Table
      historyBody.innerHTML = '';
      customerTxs.forEach(t => {
        let badge = t.status === 'PAID' 
          ? `<span class="badge bg-success">PAID</span>` 
          : (t.status === 'PARTIAL' ? `<span class="badge bg-warning text-dark">PARTIAL</span>` : `<span class="badge bg-danger">UNPAID</span>`);

        historyBody.innerHTML += `
          <tr>
            <td>${t.date}</td>
            <td class="fw-semibold">${t.product}</td>
            <td>${t.containerInfo || 'Wala'}</td>
            <td>₱${t.total.toFixed(2)}</td>
            <td class="text-success">₱${t.paid.toFixed(2)}</td>
            <td class="text-danger fw-bold">₱${t.balance.toFixed(2)}</td>
            <td>${badge}</td>
          </tr>
        `;
      });
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
      const newCustomerName = document.getElementById('payCustomerName').value.trim();
      const payAmount = parseFloat(document.getElementById('payAmountInput').value) || 0;
      const method = document.getElementById('payMethod').value;
      const tx = transactions.find(t => t.id === txId);

      if (!newCustomerName) {
        alert('Mangyaring maglagay ng pangalan ng customer.');
        return;
      }

      if (payAmount <= 0 || payAmount > tx.balance) {
        alert('Mangyaring maglagay ng tamang halaga ng ibabayad.');
        return;
      }

      // Update customer name if modified
      tx.customer = newCustomerName;

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
