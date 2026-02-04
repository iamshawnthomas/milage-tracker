<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { font-family: 'Segoe UI', Arial; padding: 20px; background: #ececec; text-align: center; }
        .card { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); max-width: 400px; margin: auto; }
        button { width: 100%; padding: 15px; margin: 10px 0; border: none; border-radius: 8px; cursor: pointer; font-size: 16px; font-weight: bold; transition: 0.3s; }
        .start { background: #28a745; color: white; }
        .stop { background: #8B2323; color: white; } /* Handyman Services Theme */
        input { width: 95%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 5px; }
        #milesDisplay { font-size: 2em; color: #333; margin: 20px 0; }
    </style>
</head>
<body>
    <div class="card">
        <h2 style="color: #8B2323;">Warranty Mileage Tracker</h2>
        <input type="text" id="purpose" placeholder="Destination / Unit #">
        <div id="milesDisplay">0.00 <span style="font-size: 0.5em;">mi</span></div>
        <p id="status">Ready to track</p>
        <button class="start" id="startBtn" onclick="startTracking()">START TRIP</button>
        <button class="stop" id="stopBtn" onclick="stopTracking()" style="display:none;">FINISH & LOG</button>
    </div>

    <script>
        const SCRIPT_URL = "YOUR_GOOGLE_SCRIPT_URL"; // PASTE YOUR WEB APP URL HERE
        let watchId = null, startPos = null, totalMiles = 0;

        function startTracking() {
            document.getElementById('status').innerText = "Tracking your route...";
            document.getElementById('startBtn').style.display = 'none';
            document.getElementById('stopBtn').style.display = 'block';
            
            navigator.geolocation.getCurrentPosition(pos => {
                startPos = pos.coords;
                watchId = navigator.geolocation.watchPosition(updatePosition, null, {enableHighAccuracy: true});
            });
        }

        function updatePosition(pos) {
            const p = 0.017453292519943295; // Math.PI / 180
            const a = 0.5 - Math.cos((pos.coords.latitude - startPos.latitude) * p)/2 + 
                      Math.cos(startPos.latitude * p) * Math.cos(pos.coords.latitude * p) * (1 - Math.cos((pos.coords.longitude - startPos.longitude) * p))/2;
            totalMiles = 7917.5 * Math.asin(Math.sqrt(a));
            document.getElementById('milesDisplay').innerHTML = totalMiles.toFixed(2) + " <span style='font-size: 0.5em;'>mi</span>";
        }

        async function stopTracking() {
            navigator.geolocation.clearWatch(watchId);
            document.getElementById('status').innerText = "Uploading to Google Sheets...";

            const payload = {
                purpose: document.getElementById('purpose').value || "Warranty Visit",
                miles: totalMiles.toFixed(2)
            };

            // Sends data to your Google Sheet backend
            fetch(SCRIPT_URL, {
                method: "POST",
                body: JSON.stringify(payload),
                mode: "no-cors" // Essential for cross-domain GitHub to Google requests
            }).then(() => {
                alert("Trip logged to Sheet!");
                location.reload();
            }).catch(err => alert("Error logging trip: " + err));
        }
    </script>
</body>
</html>
