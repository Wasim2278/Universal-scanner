<!DOCTYPE html>
<html>
<head>
  <title>Wasim Delivery Scanner</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode/dist/JsBarcode.all.min.js"></script>

  <style>
    body {
      font-family: Arial;
      background: #f3f4f6;
      margin: 0;
      padding: 20px;
    }

    .container {
      max-width: 1100px;
      margin: auto;
    }

    h1 {
      text-align: center;
      color: #111827;
    }

    .box {
      background: white;
      padding: 20px;
      border-radius: 12px;
      margin-bottom: 20px;
      box-shadow: 0 3px 12px #0002;
    }

    input, button {
      padding: 12px;
      margin: 5px;
      border-radius: 8px;
      border: 1px solid #ccc;
    }

    button {
      background: #111827;
      color: white;
      cursor: pointer;
      font-weight: bold;
    }

    .barcode-box {
      display: inline-block;
      background: white;
      border: 1px solid #ddd;
      padding: 15px;
      margin: 8px;
      text-align: center;
      border-radius: 10px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
    }

    th, td {
      padding: 10px;
      border-bottom: 1px solid #ddd;
      text-align: left;
    }

    th {
      background: #eee;
    }

    @media print {
      .no-print {
        display: none;
      }

      body {
        background: white;
      }
    }
  </style>
</head>

<body>

<div class="container">

  <div class="box no-print">
    <h1>📦 Wasim Delivery Scanner</h1>

    <p><b>Upload Excel / CSV File</b></p>

    <input type="file" id="file"
           accept=".xlsx,.xls,.csv">

    <br>

    <input type="text"
           id="holder"
           placeholder="Holder Name">

    <button onclick="generateBarcodes()">
      Generate Barcodes
    </button>

    <button onclick="window.print()">
      🖨️ Print
    </button>
  </div>

  <div class="box">
    <h3>Tracking Details</h3>

    <p>Total Shipments:
      <b id="total">0</b>
    </p>

    <table>
      <thead>
        <tr>
          <th>Tracking ID</th>
          <th>Last Updated Time</th>
          <th>Source</th>
          <th>State</th>
          <th>Last Scan By</th>
          <th>Holder Name</th>
        </tr>
      </thead>

      <tbody id="tableBody"></tbody>
    </table>
  </div>

  <div class="box">
    <h3>🏷️ Generated Barcodes</h3>

    <div id="barcodes"></div>
  </div>

</div>

<script>

let excelData = [];

document.getElementById("file")
.addEventListener("change", function(event) {

  let file = event.target.files[0];

  if (!file) return;

  let reader = new FileReader();

  reader.onload = function(e) {

    let workbook =
      XLSX.read(e.target.result, {
        type: "array"
      });

    let sheet =
      workbook.Sheets[
        workbook.SheetNames[0]
      ];

    excelData =
      XLSX.utils.sheet_to_json(sheet);

    document.getElementById("total")
      .innerText = excelData.length;

    showTable();

  };

  reader.readAsArrayBuffer(file);

});


function getValue(row, names) {

  for (let name of names) {

    if (row[name] !== undefined)
      return row[name];

  }

  return "";

}


function showTable() {

  let tbody =
    document.getElementById("tableBody");

  tbody.innerHTML = "";

  excelData.forEach(row => {

    let tracking =
      getValue(row, [
        "Tracking ID",
        "TrackingID",
        "AWB",
        "TID"
      ]);

    let time =
      getValue(row, [
        "Last Updated Time"
      ]);

    let source =
      getValue(row, [
        "Source"
      ]);

    let state =
      getValue(row, [
        "State"
      ]);

    let scan =
      getValue(row, [
        "Last Scan By"
      ]);

    let holder =
      getValue(row, [
        "Holder Name"
      ]);

    let tr =
      document.createElement("tr");

    tr.innerHTML = `
      <td>${tracking}</td>
      <td>${time}</td>
      <td>${source}</td>
      <td>${state}</td>
      <td>${scan}</td>
      <td>${holder}</td>
    `;

    tbody.appendChild(tr);

  });

}


function generateBarcodes() {

  let barcodeArea =
    document.getElementById("barcodes");

  barcodeArea.innerHTML = "";

  let holderInput =
    document.getElementById("holder").value;

  excelData.forEach((row, index) => {

    let tracking =
      getValue(row, [
        "Tracking ID",
        "TrackingID",
        "AWB",
        "TID"
      ]);

    let holder =
      holderInput ||
      getValue(row, [
        "Holder Name"
      ]);

    if (!tracking) return;

    let box =
      document.createElement("div");

    box.className =
      "barcode-box";

    box.innerHTML = `
      <b>${holder}</b>
      <br>
      <svg id="barcode${index}"></svg>
      <br>
      <small>${tracking}</small>
    `;

    barcodeArea.appendChild(box);

    JsBarcode(
      "#barcode" + index,
      String(tracking),
      {
        format: "CODE128",
        width: 2,
        height: 60,
        displayValue: true
      }
    );

  });

}

</script>

</body>
</html>
