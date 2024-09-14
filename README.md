 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to Glitch app</title>
    <style>
        /* Add your CSS styles here */
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1, h2 {
            color: #333;
        }
        #searchContainer {
            margin-bottom: 20px;
        }
        #searchInput {
            padding: 10px;
            width: calc(100% - 22px);
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        #searchButton {
            background-color: #008CBA;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            margin-top: 10px;
        }
        #searchButton:hover {
            background-color: #007bb5;
        }
        form {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background-color: #f9f9f9;
        }
        label {
            display: block;
            margin-top: 10px;
        }
        input[type="date"], 
        input[type="number"], 
        textarea {
            width: calc(100% - 22px);
            padding: 10px;
            margin-top: 5px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        textarea {
            resize: vertical;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 15px;
            margin-top: 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
        #resetButton {
            background-color: #f44336;
        }
        #resetButton:hover {
            background-color: #e53935;
        }
        #pdfButton {
            background-color: #008CBA;
        }
        #pdfButton:hover {
            background-color: #007bb5;
        }
        #invoiceTable {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        #invoiceTable, th, td {
            border: 1px solid #ddd;
        }
        th, td {
            padding: 10px;
            text-align: left;
        }
        th {
            background-color: #f4f4f4;
        }
        #totalSales {
            font-weight: bold;
            font-size: 1.2em;
        }
        .removeButton {
            background-color: #f44336;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        .removeButton:hover {
            background-color: #e53935;
        }
        .pdfButton {
            background-color: #008CBA;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
            margin-left: 5px;
        }
        .pdfButton:hover {
            background-color: #007bb5;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>
    <h1>منظومة مبيعات</h1>
    
    <!-- Search Section -->
    <div id="searchContainer">
        <input type="text" id="searchInput" placeholder="Search by product name...">
        <button id="searchButton">Search</button>
    </div>
    
    <form id="salesForm">
        <div class="form-group">
            <label for="date">Date:</label>
            <input type="date" id="date" required>
        </div>

        <div class="form-group">
            <label for="productName">Product Name:</label>
            <textarea id="productName" rows="3" required></textarea>
        </div>

        <div class="form-group">
            <label for="quantity">Quantity:</label>
            <input type="number" id="quantity" required>
        </div>

        <div class="form-group">
            <label for="unitPrice">Unit Price:</label>
            <input type="number" id="unitPrice" step="0.01" required>
        </div>

        <button type="submit">Add Invoice</button>
    </form>

    <button id="resetButton">Reset System</button>
    <button id="pdfButton">Download All PDFs</button>

    <h2>Invoices</h2>
    <table id="invoiceTable">
        <thead>
            <tr>
                <th>Date</th>
                <th>Product Name</th>
                <th>Quantity</th>
                <th>Unit Price</th>
                <th>Total Price</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
        </tbody>
    </table>
    
    <h2>Total Sales</h2>
    <p id="totalSales">0.00</p>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            loadInvoices();
        });

        document.getElementById('salesForm').addEventListener('submit', function(event) {
            event.preventDefault();
            addInvoice();
        });

        document.getElementById('resetButton').addEventListener('click', function() {
            resetSystem();
        });

        document.getElementById('pdfButton').addEventListener('click', function() {
            downloadAllPDFs();
        });

        document.getElementById('searchButton').addEventListener('click', function() {
            filterInvoices();
        });

        function addInvoice() {
            const date = document.getElementById('date').value;
            const productName = document.getElementById('productName').value;
            const quantity = parseFloat(document.getElementById('quantity').value);
            const unitPrice = parseFloat(document.getElementById('unitPrice').value);
            const totalPrice = quantity * unitPrice;

            const invoice = {
                date,
                productName,
                quantity,
                unitPrice,
                totalPrice
            };

            let invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            invoices.push(invoice);
            localStorage.setItem('invoices', JSON.stringify(invoices));

            updateTable();
            updateTotalSales();
            document.getElementById('salesForm').reset();
        }

        function loadInvoices() {
            updateTable();
            updateTotalSales();
        }

        function updateTable(invoices = null) {
            const tableBody = document.querySelector('#invoiceTable tbody');
            tableBody.innerHTML = '';

            invoices = invoices || JSON.parse(localStorage.getItem('invoices')) || [];
            invoices.forEach((invoice, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${invoice.date}</td>
                    <td>${invoice.productName}</td>
                    <td>${invoice.quantity}</td>
                    <td>${invoice.unitPrice.toFixed(2)}</td>
                    <td>${invoice.totalPrice.toFixed(2)}</td>
                    <td>
                        <button class="removeButton" data-index="${index}">Remove</button>
                        <button class="pdfButton" data-index="${index}">Download PDF</button>
                    </td>
                `;
                tableBody.appendChild(row);
            });

            // Attach event listeners to "Remove" and "Download PDF" buttons
            document.querySelectorAll('.removeButton').forEach(button => {
                button.addEventListener('click', function() {
                    removeInvoice(parseInt(this.getAttribute('data-index')));
                });
            });

            document.querySelectorAll('.pdfButton').forEach(button => {
                button.addEventListener('click', function() {
                    generatePDF(parseInt(this.getAttribute('data-index')));
                });
            });
        }

        function updateTotalSales() {
            let invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            const totalSales = invoices.reduce((sum, invoice) => sum + invoice.totalPrice, 0);
            document.getElementById('totalSales').textContent = totalSales.toFixed(2);
        }

        function resetSystem() {
            localStorage.removeItem('invoices');
            updateTable();
            updateTotalSales();
        }

        function generatePDF(index = null) {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'mm', 'a4'); // Portrait orientation

            const margin = 15;
            const pageWidth = doc.internal.pageSize.width;

            if (index === null) {
                // If no index is provided, generate a PDF for all invoices
                const title = 'Atlas Libya Company';
                doc.setFontSize(16);
                doc.setFont('Arial', 'bold');
                doc.text(title, pageWidth / 2, margin + 10, { align: 'center' });

                const subtitle = 'Invoices';
                doc.setFontSize(14);
                doc.setFont('Arial', 'bold');
                doc.text(subtitle, pageWidth / 2, margin + 20, { align: 'center' });

                const columnHeaders = ['Date', 'Product Name', 'Quantity', 'Unit Price', 'Total Price'];
                const columnWidths = [30, 80, 30, 30, 30];
                let x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2;
                let y = margin + 30;

                doc.setFontSize(12);
                doc.setFont('Arial', 'bold');
                columnHeaders.forEach((header, index) => {
                    const cellWidth = columnWidths[index];
                    doc.rect(x, y, cellWidth, 10);
                    doc.text(header, x + cellWidth / 2, y + 7, { align: 'center' });
                    x += columnWidths[index];
                });

                y += 10; // Move to the next line after headers
                x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2; // Reset x to start position

                // Table rows
                const tableRows = JSON.parse(localStorage.getItem('invoices')) || [];
                tableRows.forEach(row => {
                    doc.setFont('Arial', 'normal');
                    const rowData = [
                        row.date,
                        row.productName,
                        row.quantity.toString(),
                        row.unitPrice.toFixed(2),
                        row.totalPrice.toFixed(2)
                    ];

                    rowData.forEach((data, index) => {
                        const cellWidth = columnWidths[index];
                        doc.rect(x, y, cellWidth, 10);
                        doc.text(data, x + cellWidth / 2, y + 7, { align: 'center' });
                        x += cellWidth;
                    });
                    
                    y += 10; // Move to the next row
                    x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2; // Reset x to start position
                });

                // Footer with total sales
                y += 10;
                doc.setFontSize(12);
                doc.setFont('Arial', 'bold');
                const totalText = `Total Sales: ${document.getElementById('totalSales').textContent}`;
                doc.text(totalText, pageWidth / 2, y, { align: 'center' });

                // Save the PDF
                doc.save('invoice.pdf');
            } else {
                // Generate a PDF for a single invoice
                const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
                const invoice = invoices[index];

                doc.setFontSize(16);
                doc.setFont('Arial', 'bold');
                doc.text('Atlas Libya Company', pageWidth / 2, margin + 10, { align: 'center' });

                doc.setFontSize(14);
                doc.setFont('Arial', 'bold');
                doc.text('Invoice Details', pageWidth / 2, margin + 20, { align: 'center' });

                // Add invoice details
                const details = [
                    `Date: ${invoice.date}`,
                    `Product Name: ${invoice.productName}`,
                    `Quantity: ${invoice.quantity}`,
                    `Unit Price: ${invoice.unitPrice.toFixed(2)}`,
                    `Total Price: ${invoice.totalPrice.toFixed(2)}`
                ];

                const boxWidth = 70;
                const boxHeight = 10;
                const startX = margin;
                const startY = margin + 30;

                doc.setFontSize(12);
                doc.setFont('Arial', 'normal');

                details.forEach((detail, index) => {
                    const row = Math.floor(index / 2);
                    const col = index % 2;
                    const x = startX + (col * (boxWidth + 5));
                    const y = startY + (row * (boxHeight + 5));

                    doc.rect(x, y, boxWidth, boxHeight);
                    doc.text(detail, x + 5, y + 7);
                });

                // Signature area
                const signatureY = startY + ((details.length + 1) * (boxHeight + 5)) + 10;
                doc.text('Signature: ___________________________', margin, signatureY);

                // Save the PDF
                doc.save('invoice-details.pdf');
            }
        }

        function removeInvoice(index) {
            let invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            invoices.splice(index, 1);
            localStorage.setItem('invoices', JSON.stringify(invoices));
            updateTable();
            updateTotalSales();
        }

        function filterInvoices() {
            const searchTerm = document.getElementById('searchInput').value.toLowerCase();
            const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            const filteredInvoices = invoices.filter(invoice =>
                invoice.productName.toLowerCase().includes(searchTerm)
            );
            updateTable(filteredInvoices);

            // Update PDF button functionality to only download filtered invoices
            document.getElementById('pdfButton').removeEventListener('click', downloadAllPDFs);
            document.getElementById('pdfButton').addEventListener('click', () => generatePDFForFiltered(filteredInvoices));
        }

        function generatePDFForFiltered(filteredInvoices) {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'mm', 'a4'); // Portrait orientation

            const margin = 15;
            const pageWidth = doc.internal.pageSize.width;

            const title = 'Atlas Libya Company';
            doc.setFontSize(16);
            doc.setFont('Arial', 'bold');
            doc.text(title, pageWidth / 2, margin + 10, { align: 'center' });

            const subtitle = 'Filtered Invoices';
            doc.setFontSize(14);
            doc.setFont('Arial', 'bold');
            doc.text(subtitle, pageWidth / 2, margin + 20, { align: 'center' });

            const columnHeaders = ['Date', 'Product Name', 'Quantity', 'Unit Price', 'Total Price'];
            const columnWidths = [30, 80, 30, 30, 30];
            let x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2;
            let y = margin + 30;

            doc.setFontSize(12);
            doc.setFont('Arial', 'bold');
            columnHeaders.forEach((header, index) => {
                const cellWidth = columnWidths[index];
                doc.rect(x, y, cellWidth, 10);
                doc.text(header, x + cellWidth / 2, y + 7, { align: 'center' });
                x += columnWidths[index];
            });

            y += 10; // Move to the next line after headers
            x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2; // Reset x to start position

            // Table rows
            filteredInvoices.forEach(row => {
                doc.setFont('Arial', 'normal');
                const rowData = [
                    row.date,
                    row.productName,
                    row.quantity.toString(),
                    row.unitPrice.toFixed(2),
                    row.totalPrice.toFixed(2)
                ];

                rowData.forEach((data, index) => {
                    const cellWidth = columnWidths[index];
                    doc.rect(x, y, cellWidth, 10);
                    doc.text(data, x + cellWidth / 2, y + 7, { align: 'center' });
                    x += cellWidth;
                });
                
                y += 10; // Move to the next row
                x = (pageWidth - columnWidths.reduce((a, b) => a + b, 0)) / 2; // Reset x to start position
            });

            // Footer with total sales
            y += 10;
            doc.setFontSize(12);
            doc.setFont('Arial', 'bold');
            const totalText = `Total Sales: ${filteredInvoices.reduce((sum, invoice) => sum + invoice.totalPrice, 0).toFixed(2)}`;
            doc.text(totalText, pageWidth / 2, y, { align: 'center' });

            // Save the PDF
            doc.save('filtered-invoices.pdf');
        }

        function downloadAllPDFs() {
            generatePDF(); // Existing function to generate all PDFs
        }
    </script>
</body>
</html>
