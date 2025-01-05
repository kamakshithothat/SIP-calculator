<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SIP Return Calculator</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f8f9fa;
        }
        header {
            background-color: #007bff;
            color: white;
            text-align: center;
            padding: 1rem;
        }
        main {
            max-width: 800px;
            margin: 2rem auto;
            background: white;
            padding: 2rem;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .form-group {
            margin-bottom: 1.5rem;
        }
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        select, input {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 0.75rem 1.5rem;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        button:hover {
            background-color: #0056b3;
        }
        .chart-container {
            margin-top: 2rem;
        }
        .result-container {
            margin-top: 2rem;
            background: #f1f1f1;
            padding: 1rem;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <header>
        <h1>SIP Return Calculator</h1>
    </header>
    <main>
        <form id="sip-form">
            <div class="form-group">
                <label for="fund-house">Mutual Fund House</label>
                <select id="fund-house" name="fundHouse" required>
                    <option value="">Select Fund House</option>
                    <option value="HDFC">HDFC Mutual Fund</option>
                    <option value="SBI">SBI Mutual Fund</option>
                    <option value="ICICI">ICICI Prudential Mutual Fund</option>
                    <option value="Axis">Axis Mutual Fund</option>
                    <option value="All India">All India Mutual Funds</option>
                </select>
            </div>
            <div class="form-group">
                <label for="fund-name">Mutual Fund Name</label>
                <select id="fund-name" name="fundName" required>
                    <option value="">Select a Fund House First</option>
                </select>
            </div>
            <div class="form-group">
                <label for="investment-period">Investment Time Period (in years)</label>
                <input type="number" id="investment-period" name="investmentPeriod" min="1" placeholder="Enter years" required>
            </div>
            <div class="form-group">
                <label for="return-rate">Expected Return Rate per Annum (in %)</label>
                <input type="number" id="return-rate" name="returnRate" min="1" step="0.1" placeholder="Enter rate in %" required>
            </div>
            <button type="button" onclick="calculateSIP()">Calculate</button>
        </form>

        <div class="result-container" id="result-container" style="display: none;">
            <h3>Results:</h3>
            <p id="invested-amount">Total Invested Amount: ₹0</p>
            <p id="total-value">Expected Total Value: ₹0</p>
        </div>

        <div class="chart-container">
            <canvas id="sip-chart" width="400" height="200"></canvas>
        </div>
    </main>

    <script>
        const fundHouseToFunds = {
            HDFC: ["HDFC Equity Fund", "HDFC Balanced Advantage Fund", "HDFC Small Cap Fund"],
            SBI: ["SBI Bluechip Fund", "SBI Magnum Multicap Fund", "SBI Equity Hybrid Fund"],
            ICICI: ["ICICI Prudential Value Discovery Fund", "ICICI Prudential Balanced Advantage Fund", "ICICI Prudential Technology Fund"],
            Axis: ["Axis Bluechip Fund", "Axis Midcap Fund", "Axis Small Cap Fund"],
            "All India": ["All India Equity Fund", "All India Balanced Fund", "All India Small Cap Fund"]
        };

        document.getElementById("fund-house").addEventListener("change", function () {
            const fundHouse = this.value;
            const fundNameSelect = document.getElementById("fund-name");
            fundNameSelect.innerHTML = "<option value=''>Select Mutual Fund</option>";

            if (fundHouse && fundHouseToFunds[fundHouse]) {
                fundHouseToFunds[fundHouse].forEach(fund => {
                    const option = document.createElement("option");
                    option.value = fund;
                    option.textContent = fund;
                    fundNameSelect.appendChild(option);
                });
            }
        });

        function calculateSIP() {
            const investmentPeriod = parseInt(document.getElementById('investment-period').value);
            const returnRate = parseFloat(document.getElementById('return-rate').value) / 100;
            const monthlyInvestment = 10000; // Fixed monthly SIP

            let investedAmount = 0;
            let totalValue = 0;
            let yearlyData = [];

            for (let year = 1; year <= investmentPeriod; year++) {
                investedAmount += monthlyInvestment * 12;
                totalValue = investedAmount * Math.pow(1 + returnRate, year / investmentPeriod);
                yearlyData.push({ year: year, invested: investedAmount, value: totalValue });
            }

            document.getElementById('invested-amount').textContent = `Total Invested Amount: ₹${investedAmount.toFixed(2)}`;
            document.getElementById('total-value').textContent = `Expected Total Value: ₹${totalValue.toFixed(2)}`;
            document.getElementById('result-container').style.display = 'block';

            renderChart(yearlyData);
        }

        function renderChart(data) {
            const ctx = document.getElementById('sip-chart').getContext('2d');

            const labels = data.map(entry => `Year ${entry.year}`);
            const investedData = data.map(entry => entry.invested);
            const valueData = data.map(entry => entry.value);

            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'Invested Amount',
                            data: investedData,
                            borderColor: '#ff6384',
                            fill: false,
                        },
                        {
                            label: 'Total Value',
                            data: valueData,
                            borderColor: '#36a2eb',
                            fill: false,
                        },
                    ],
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        },
                    },
                    scales: {
                        y: {
                            title: {
                                display: true,
                                text: 'Amount (₹)',
                            },
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Years',
                            },
                        },
                    },
                },
            });
        }
    </script>
</body>
</html>
