<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daily Cashier Sales & Audit Sheet</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: { 50: '#f0fdf4', 100: '#dcfce7', 500: '#22c55e', 600: '#16a34a', 700: '#15803d' }
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen pb-12">

    <!-- Header -->
    <header class="bg-slate-900 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 py-4 flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="flex items-center gap-3">
                <div class="bg-emerald-500 text-slate-900 p-2.5 rounded-xl font-black text-xl shadow">
                    <i class="fa-solid fa-cash-register"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold tracking-tight">Daily Cashier Sales & Audit</h1>
                    <p class="text-xs text-slate-400">Shift Reconciliation & Denomination Counter</p>
                </div>
            </div>
            <div class="flex items-center gap-3">
                <button onclick="resetForm()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 text-sm font-medium rounded-lg transition border border-slate-700">
                    <i class="fa-solid fa-rotate-right mr-1.5"></i> Reset Form
                </button>
                <button onclick="window.print()" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-500 text-white text-sm font-semibold rounded-lg shadow transition">
                    <i class="fa-solid fa-print mr-1.5"></i> Print / Save PDF
                </button>
            </div>
        </div>
    </header>

    <main class="max-w-7xl mx-auto px-4 mt-6 grid grid-cols-1 lg:grid-cols-3 gap-6">

        <!-- Left 2 Columns: Denominations & Inputs -->
        <div class="lg:col-span-2 space-y-6">

            <!-- Shift Information Card -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-5">
                <h2 class="text-sm font-bold uppercase tracking-wider text-slate-400 mb-3">Shift Details</h2>
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Date</label>
                        <input type="date" id="auditDate" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Cashier Name</label>
                        <input type="text" id="cashierName" placeholder="Enter name" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Shift / Branch</label>
                        <input type="text" id="shiftInfo" placeholder="e.g. Shift 1 / Branch A" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                </div>
            </div>

            <!-- Currency Denominations Table -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
                <div class="bg-slate-100 px-5 py-3 border-b border-slate-200 flex justify-between items-center">
                    <h2 class="font-bold text-slate-700 flex items-center gap-2">
                        <i class="fa-solid fa-coins text-emerald-600"></i> Cash Denominations
                    </h2>
                    <span class="text-xs text-slate-500 font-medium">Auto-calculates total count</span>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-50 text-slate-500 text-xs uppercase tracking-wider border-b border-slate-200">
                                <th class="py-3 px-4 font-semibold">Denomination</th>
                                <th class="py-3 px-4 font-semibold text-center">Multiplier</th>
                                <th class="py-3 px-4 font-semibold text-center">Quantity (PCS)</th>
                                <th class="py-3 px-4 font-semibold text-right">Total Amount (₱)</th>
                            </tr>
                        </thead>
                        <tbody id="denomRows" class="divide-y divide-slate-100 text-sm">
                            <!-- Populated dynamically via JS -->
                        </tbody>
                        <tfoot>
                            <tr class="bg-slate-50 font-bold border-t border-slate-200">
                                <td colspan="3" class="py-3 px-4 text-right text-slate-700">Total Counted Cash:</td>
                                <td id="totalCountedDisplay" class="py-3 px-4 text-right text-emerald-600 text-base">₱0.00</td>
                            </tr>
                        </tfoot>
                    </table>
                </div>
            </div>

        </div>

        <!-- Right Column: Audit, Deductions & Summary -->
        <div class="space-y-6">

            <!-- Cash Audit & Deductions Card -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-5 space-y-4">
                <h2 class="font-bold text-slate-700 flex items-center gap-2 border-b border-slate-100 pb-3">
                    <i class="fa-solid fa-calculator text-emerald-600"></i> Cash Audit & Deductions
                </h2>

                <!-- Pondo / Change Fund Input (Bagong dagdag dito sa tabi) -->
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1 flex items-center justify-between">
                        <span>Pondo / Change Fund</span>
                        <span class="text-slate-400 font-normal">Starting Fund</span>
                    </label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400 text-sm">₱</span>
                        <input type="number" id="inputPondo" value="0" min="0" step="any" oninput="calculateTotals()" 
                               class="w-full pl-8 pr-3 py-2 bg-slate-50 border border-slate-300 rounded-lg text-sm font-semibold text-slate-700 focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Target / Expected Cash (System Sales)</label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400 text-sm">₱</span>
                        <input type="number" id="targetSales" value="0" min="0" step="any" oninput="calculateTotals()" 
                               class="w-full pl-8 pr-3 py-2 bg-slate-50 border border-slate-300 rounded-lg text-sm font-semibold text-slate-700 focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">GCash / Online / Card Payments</label>
                    <div class="relative">
                        <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400 text-sm">₱</span>
                        <input type="number" id="nonCashPayments" value="0" min="0" step="any" oninput="calculateTotals()" 
                               class="w-full pl-8 pr-3 py-2 bg-slate-50 border border-slate-300 rounded-lg text-sm font-semibold text-slate-700 focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </div>
                </div>

                <div class="border-t border-slate-100 pt-3 space-y-2">
                    <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Deductions / Expenses</p>
                    <div id="expensesContainer" class="space-y-2">
                        <!-- Dynamic Expense Rows will appear here -->
                    </div>
                    <button onclick="addExpenseRow()" class="text-xs text-emerald-600 hover:text-emerald-700 font-semibold flex items-center gap-1 mt-1">
                        <i class="fa-solid fa-plus-circle"></i> Add Deduction / Expense
                    </button>
                </div>
            </div>

            <!-- Summary / Variance Result Card -->
            <div class="bg-slate-900 text-white rounded-2xl shadow-lg p-5 space-y-4">
                <h2 class="text-sm font-bold uppercase tracking-wider text-slate-400 border-b border-slate-800 pb-2">Shift Summary</h2>
                
                <div class="space-y-2 text-sm">
                    <div class="flex justify-between text-slate-300">
                        <span>Total Counted Cash:</span>
                        <span id="sumTotalCounted" class="font-medium">₱0.00</span>
                    </div>
                    <div class="flex justify-between text-slate-300">
                        <span>Minus Pondo (Change Fund):</span>
                        <span id="sumPondo" class="font-medium text-amber-400">-₱0.00</span>
                    </div>
                    <div class="flex justify-between text-slate-300">
                        <span>Minus Total Deductions:</span>
                        <span id="sumDeductions" class="font-medium text-rose-400">-₱0.00</span>
                    </div>
                    <div class="flex justify-between text-slate-300 border-t border-slate-800 pt-2">
                        <span>Net Cash Drawer:</span>
                        <span id="sumNetDrawer" class="font-semibold text-white">₱0.00</span>
                    </div>
                    <div class="flex justify-between text-slate-300">
                        <span>Expected Cash (Target):</span>
                        <span id="sumTarget" class="font-medium">₱0.00</span>
                    </div>
                </div>

                <div id="varianceBox" class="p-3.5 rounded-xl bg-slate-800 border border-slate-700 text-center mt-3">
                    <p class="text-xs text-slate-400 uppercase font-semibold">Variance / Shortage / Overage</p>
                    <p id="varianceValue" class="text-2xl font-bold mt-1">₱0.00</p>
                    <p id="varianceStatus" class="text-xs mt-1 text-slate-400 font-medium">Balanced</p>
                </div>
            </div>

        </div>

    </main>

    <script>
        // Denominations List
        const denominations = [
            1000, 500, 200, 100, 50, 20, 10, 5, 1, 0.25
        ];

        function initApp() {
            const tbody = document.getElementById('denomRows');
            tbody.innerHTML = '';

            denominations.forEach((denom) => {
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-slate-50/50 transition';
                
                const label = denom >= 1 ? `₱${denom}` : `${denom * 100}¢`;
                
                tr.innerHTML = `
                    <td class="py-3 px-4 font-medium text-slate-700">${label}</td>
                    <td class="py-3 px-4 text-center text-slate-400 text-xs">x ${denom}</td>
                    <td class="py-3 px-4 text-center">
                        <input type="number" min="0" value="" placeholder="0" data-denom="${denom}" oninput="calculateTotals()"
                            class="denom-input w-20 text-center bg-slate-50 border border-slate-300 rounded-md py-1 text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </td>
                    <td class="py-3 px-4 text-right font-medium text-slate-700 row-total" data-denom-val="${denom}">₱0.00</td>
                `;
                tbody.appendChild(tr);
            });

            // Add default sample deduction row
            addExpenseRow('Initial Float Change', 0);
            calculateTotals();
        }

        function addExpenseRow(desc = '', amount = 0) {
            const container = document.getElementById('expensesContainer');
            const div = document.createElement('div');
            div.className = 'flex items-center gap-2 expense-row';
            div.innerHTML = `
                <input type="text" placeholder="Expense / Reason" value="${desc}" class="expense-desc flex-1 bg-slate-50 border border-slate-300 rounded-lg px-2.5 py-1.5 text-xs focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                <input type="number" placeholder="0.00" value="${amount}" min="0" step="any" oninput="calculateTotals()" class="expense-amount w-28 bg-slate-50 border border-slate-300 rounded-lg px-2.5 py-1.5 text-xs text-right focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                <button onclick="this.parentElement.remove(); calculateTotals();" class="text-slate-400 hover:text-rose-500 p-1">
                    <i class="fa-solid fa-trash-can text-xs"></i>
                </button>
            `;
            container.appendChild(div);
            calculateTotals();
        }

        function formatCurrency(amount) {
            return '₱' + amount.toLocaleString('en-PH', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
        }

        function calculateTotals() {
            let totalCounted = 0;

            // Calculate Denominations
            document.querySelectorAll('.denom-input').forEach(input => {
                const denom = parseFloat(input.dataset.denom);
                const qty = parseInt(input.value) || 0;
                const subtotal = denom * qty;
                totalCounted += subtotal;

                const rowTotalEl = input.closest('tr').querySelector('.row-total');
                rowTotalEl.textContent = formatCurrency(subtotal);
            });

            document.getElementById('totalCountedDisplay').textContent = formatCurrency(totalCounted);
            document.getElementById('sumTotalCounted').textContent = formatCurrency(totalCounted);

            // Get Pondo & Target
            const pondo = parseFloat(document.getElementById('inputPondo').value) || 0;
            const targetSales = parseFloat(document.getElementById('targetSales').value) || 0;
            const nonCash = parseFloat(document.getElementById('nonCashPayments').value) || 0;

            document.getElementById('sumPondo').textContent = '-' + formatCurrency(pondo);

            // Calculate Total Deductions/Expenses
            let totalDeductions = 0;
            document.querySelectorAll('.expense-amount').forEach(input => {
                totalDeductions += parseFloat(input.value) || 0;
            });

            document.getElementById('sumDeductions').textContent = '-' + formatCurrency(totalDeductions);

            // Net Cash Drawer = Total Counted - Pondo - Deductions
            const netDrawer = totalCounted - pondo - totalDeductions;
            document.getElementById('sumNetDrawer').textContent = formatCurrency(netDrawer);

            // Expected Cash (Target Cash Sales minus Non-Cash/GCash payments)
            const expectedCash = targetSales - nonCash;
            document.getElementById('sumTarget').textContent = formatCurrency(expectedCash);

            // Variance = Net Cash Drawer - Expected Cash
            const variance = netDrawer - expectedCash;
            
            const varianceValEl = document.getElementById('varianceValue');
            const varianceStatusEl = document.getElementById('varianceStatus');
            const varianceBox = document.getElementById('varianceBox');

            varianceValEl.textContent = (variance >= 0 ? '+' : '') + formatCurrency(variance);

            if (Math.abs(variance) < 0.01) {
                varianceBox.className = 'p-3.5 rounded-xl bg-emerald-950/60 border border-emerald-800 text-center mt-3';
                varianceValEl.className = 'text-2xl font-bold mt-1 text-emerald-400';
                varianceStatusEl.textContent = 'Balanced (Exact)';
                varianceStatusEl.className = 'text-xs mt-1 text-emerald-300 font-medium';
            } else if (variance > 0) {
                varianceBox.className = 'p-3.5 rounded-xl bg-blue-950/60 border border-blue-800 text-center mt-3';
                varianceValEl.className = 'text-2xl font-bold mt-1 text-blue-400';
                varianceStatusEl.textContent = 'Overage (Sobri)';
                varianceStatusEl.className = 'text-xs mt-1 text-blue-300 font-medium';
            } else {
                varianceBox.className = 'p-3.5 rounded-xl bg-rose-950/60 border border-rose-800 text-center mt-3';
                varianceValEl.className = 'text-2xl font-bold mt-1 text-rose-400';
                varianceStatusEl.textContent = 'Shortage (Kulang)';
                varianceStatusEl.className = 'text-xs mt-1 text-rose-300 font-medium';
            }
        }

        function resetForm() {
            if (confirm('Gusto mo bang i-reset lahat ng laman ng form?')) {
                document.getElementById('auditDate').value = '';
                document.getElementById('cashierName').value = '';
                document.getElementById('shiftInfo').value = '';
                document.getElementById('inputPondo').value = '0';
                document.getElementById('targetSales').value = '0';
                document.getElementById('nonCashPayments').value = '0';
                document.querySelectorAll('.denom-input').forEach(i => i.value = '');
                document.getElementById('expensesContainer.innerHTML = ""');
                addExpenseRow();
                calculateTotals();
            }
        }

        // Initialize on load
        window.onload = initApp;
    </script>
</body>
</html>
