<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Prediction</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            color: #333;
            margin: 0;
            padding: 20px;
        }

        #mainApp {
            max-width: 600px;
            margin: 0 auto;
            background: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        h1, h2 {
            text-align: center;
        }

        #stats, #currentPrediction {
            margin-bottom: 20px;
        }

        #history {
            margin-top: 20px;
        }

        .history-item {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin-bottom: 10px;
            background: #f9f9f9;
        }

        .win {
            color: green;
        }

        .loss {
            color: red;
        }

        .pending {
            color: orange;
        }
    </style>
</head>
<body>
    <div id="mainApp">
        <h1>BD_SHANTO Paid Hack</h1>
        <div id="stats">
            <p>Total Wins: <span id="totalWins">0</span></p>
            <p>Total Losses: <span id="totalLosses">0</span></p>
            <p>Accuracy: <span id="accuracy">0%</span></p>
        </div>
        <div id="currentPrediction">
            <p>Current Period: <span id="currentPeriod">-</span></p>
            <p>Prediction : <span id="prediction">-</span></p>
        </div>
        <div id="history">
            <h2>History</h2>
        </div>
    </div>

    <script>
        let historyData = [];
        let totalWins = 0;
        let totalLosses = 0;
        let totalPredictions = 0;
        let correctPredictions = 0;
        let currentPeriod = null;
        let lastFetchedPeriod = null;

        // Function to fetch game result
        async function fetchGameResult() {
            try {
                const payload = {
                    pageSize: 10,
                    pageNo: 1,
                    typeId: 1,
                    language: 0,
                    random: "4a0522c6ecd8410496260e686be2a57c",
                    signature: "334B5E70A0C9B8918B0B15E517E2069C",
                    timestamp: Math.floor(Date.now() / 1000)
                };

                let response = await fetch("https://api.bdg88zf.com/api/webapi/GetNoaverageEmerdList", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`HTTP error! Status: ${response.status}`);
                }

                let data = await response.json();
                let latestResult = data?.data?.list?.[0];
                if (latestResult) {
                    return { period: latestResult.issueNumber, result: latestResult.number };
                } else {
                    throw new Error("No data found in the API response");
                }
            } catch (error) {
                console.error("Error fetching game result:", error);
                return null;
            }
        }

        // Algorithm 1: Trend Analysis
        function trendAnalysis(history) {
            let bigCount = history.filter(item => item.result >= 5).length;
            let smallCount = history.filter(item => item.result < 5).length;
            return bigCount > smallCount ? "BIG" : "SMALL";
        }

        // Auto-predict function
        function autoPredict(actualResult) {
            let prediction = trendAnalysis(historyData);
            return { type: prediction };
        }

        // Function to update prediction
        async function updatePrediction() {
            let apiResult = await fetchGameResult();

            if (apiResult && apiResult.period !== lastFetchedPeriod) {
                lastFetchedPeriod = apiResult.period;
                currentPeriod = (BigInt(apiResult.period) + 1n).toString();

                let prediction = autoPredict(apiResult.result);

                document.getElementById("currentPeriod").innerText = currentPeriod;
                document.getElementById("prediction").innerText = prediction.type;

                historyData.unshift({ period: currentPeriod, result: apiResult.result, prediction: prediction.type, resultStatus: "Pending" });
                updateHistory();
                checkWinLoss(apiResult);
            }
        }

        // Function to check win/loss
        async function checkWinLoss(apiResult) {
            if (!apiResult) return;

            historyData.forEach(item => {
                if (item.period === apiResult.period) {
                    let actualResult = apiResult.result >= 5 ? "SMALL" : "BIG";
                    item.resultStatus = (item.prediction === actualResult) ? "LOSS" : "WIN";
                }
            });

            updateStats(apiResult);
            updateHistory();
        }

        // Function to update stats
        function updateStats(apiResult) {
            totalWins = historyData.filter(item => item.resultStatus === "WIN").length;
            totalLosses = historyData.filter(item => item.resultStatus === "LOSS").length;
            totalPredictions = historyData.length;
            correctPredictions = totalWins;

            document.getElementById("totalWins").innerText = totalWins;
            document.getElementById("totalLosses").innerText = totalLosses;
            document.getElementById("accuracy").innerText = calculateConfidence();
        }

        // Function to calculate accuracy
        function calculateConfidence() {
            let confidence = (correctPredictions / totalPredictions) * 100 || 0;
            return confidence.toFixed(2) + '%';
        }

        // Function to update history
        function updateHistory() {
            let historyDiv = document.getElementById("history");
            historyDiv.innerHTML = "";

            historyData.forEach(item => {
                let resultClass = item.resultStatus === "WIN" ? "win" : (item.resultStatus === "LOSS" ? "loss" : "pending");
                let resultText = item.resultStatus === "WIN" ? "☑ WIN" : (item.resultStatus === "LOSS" ? "🚫 LOSS" : "📶 Pending");

                historyDiv.innerHTML += `
                    <div class="history-item">
                        ⏳ <strong>Period:</strong> ${item.period}<br>
                        ⭐ <strong>Prediction:</strong> ${item.prediction}<br>
                        <span class="${resultClass}">${resultText}</span>
                    </div>`;
            });
        }

        // Update prediction every second
        setInterval(updatePrediction, 1000);
    </script>
</body>
</html>
