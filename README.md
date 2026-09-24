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
