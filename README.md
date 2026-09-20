<!-- ================= 3.5 CUSTOMER ORDER LOOKUP TAB ================= -->
      <div class="tab-pane fade" id="search-content">
        <div class="card p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="card-title text-primary m-0"><i class="fa-solid fa-magnifying-glass me-2"></i>Order & Daily Audit Lookup</h4>
            <button class="btn btn-outline-secondary" onclick="window.print()">
              <i class="fa-solid fa-print me-1"></i> Print Record
            </button>
          </div>
          
          <!-- SEARCH MODE TABS / SWITCHER -->
          <ul class="nav nav-tabs mb-3" id="lookupTabs" role="tablist">
            <li class="nav-item" role="presentation">
              <button class="nav-link active fw-bold" id="by-customer-tab" data-bs-toggle="tab" data-bs-target="#searchByCustomer" type="button" role="tab">Search by Customer</button>
            </li>
            <li class="nav-item" role="presentation">
              <button class="nav-link fw-bold" id="by-date-tab" data-bs-toggle="tab" data-bs-target="#searchByDate" type="button" role="tab">Search by Date (Buong Araw Audit)</button>
            </li>
          </ul>

          <div class="tab-content">
            <!-- 1. SEARCH BY CUSTOMER -->
            <div class="tab-pane fade show active" id="searchByCustomer">
              <div class="row g-2 mb-4">
                <div class="col-md-9">
                  <input type="text" id="searchCustomerInput" class="form-control form-control-lg" placeholder="I-type ang pangalan ng Customer (e.g. Juan Dela Cruz)">
                </div>
                <div class="col-md-3">
                  <button class="btn btn-primary btn-lg w-100 fw-bold" onclick="searchCustomerOrder()">
                    <i class="fa-solid fa-search me-2"></i>Search Customer
                  </button>
                </div>
              </div>

              <!-- Result Display Area (Customer) -->
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
                        <p class="mb-1 text-muted small fw-bold">ITEMS / PRODUCTS & DESCRIPTION BOUGHT:</p>
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
                        <th>Products & Description</th>
                        <th>Container</th>
                        <th>Total (₱)</th>
                        <th>Paid (₱)</th>
                        <th>Balance (₱)</th>
                        <th>Net Profit (₱)</th>
                        <th>Status</th>
                      </tr>
                    </thead>
                    <tbody id="customerHistoryBody"></tbody>
                  </table>
                </div>
              </div>

              <div id="noCustomerFound" class="alert alert-warning text-center p-3 d-none">
                <i class="fa-solid fa-triangle-exclamation me-2"></i> Walang nahanap na record para sa customer na ito.
              </div>
            </div>

            <!-- 2. SEARCH BY DATE (BUONG ARAW AUDIT, INVENTORY, UTANG, NET PROFIT) -->
            <div class="tab-pane fade" id="searchByDate">
              <div class="row g-2 mb-4">
                <div class="col-md-9">
                  <input type="date" id="searchDateInput" class="form-control form-control-lg">
                </div>
                <div class="col-md-3">
                  <button class="btn btn-success btn-lg w-100 fw-bold" onclick="searchByDateRecord()">
                    <i class="fa-solid fa-calendar-check me-2"></i>Search Date Audit
                  </button>
                </div>
              </div>

              <!-- Date Search Results Container -->
              <div id="searchDateResultContainer" style="display: none;">
                <div class="row g-3 mb-4">
                  <div class="col-md-3">
                    <div class="card p-3 stat-card bg-light">
                      <span class="text-muted small fw-bold">GROSS SALES</span>
                      <h5 class="text-primary mt-1 mb-0" id="lookupDateSales">₱0.00</h5>
                    </div>
                  </div>
                  <div class="col-md-3">
                    <div class="card p-3 stat-card bg-light" style="border-left-color: #2e7d32;">
                      <span class="text-muted small fw-bold">TOTAL COLLECTIONS</span>
                      <h5 class="text-success mt-1 mb-0" id="lookupDateCollected">₱0.00</h5>
                    </div>
                  </div>
                  <div class="col-md-3">
                    <div class="card p-3 stat-card bg-light" style="border-left-color: #c62828;">
                      <span class="text-muted small fw-bold">TOTAL UTANG (Balance)</span>
                      <h5 class="text-danger mt-1 mb-0" id="lookupDateUtang">₱0.00</h5>
                    </div>
                  </div>
                  <div class="col-md-3">
                    <div class="card p-3 stat-card bg-light" style="border-left-color: #00897b;">
                      <span class="text-muted small fw-bold">NET PROFIT</span>
                      <h5 class="text-success fw-bold mt-1 mb-0" id="lookupDateNetProfit">₱0.00</h5>
                    </div>
                  </div>
                </div>

                <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-list me-2"></i>Mga Transaksyon sa Petsang Ito</h6>
                <div class="table-responsive mb-4">
                  <table class="table table-bordered table-hover align-middle bg-white">
                    <thead class="table-dark">
                      <tr>
                        <th>Customer</th>
                        <th>Location</th>
                        <th>Products & Description</th>
                        <th>Container</th>
                        <th>Total (₱)</th>
                        <th>Paid (₱)</th>
                        <th>Balance (₱)</th>
                        <th>Net Profit (₱)</th>
                        <th>Status</th>
                      </tr>
                    </thead>
                    <tbody id="lookupDateTransactionsBody"></tbody>
                  </table>
                </div>

                <h6 class="fw-bold text-secondary mb-3"><i class="fa-solid fa-boxes-stacked me-2"></i>Inventory & Products na Na-encode / Na-galaw sa Araw na Ito</h6>
                <div class="table-responsive">
                  <table class="table table-bordered table-hover align-middle bg-white">
                    <thead class="table-secondary">
                      <tr>
                        <th>Product Name</th>
                        <th>Description / Box Info</th>
                        <th>Cost / Unit (₱)</th>
                        <th>Price / Unit (₱)</th>
                        <th>Total Sold Qty</th>
                        <th>Total Sales (₱)</th>
                      </tr>
                    </thead>
                    <tbody id="lookupDateInventoryBody"></tbody>
                  </table>
                </div>
              </div>

              <div id="noDateFound" class="alert alert-warning text-center p-3 d-none">
                <i class="fa-solid fa-triangle-exclamation me-2"></i> Walang nahanap na transaksyon o record sa petsang ito.
              </div>
            </div>
          </div>

        </div>
      </div>
