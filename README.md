<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Parking System</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            text-align: center;
            width: 400px;
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
        }

        #login-screen input {
            width: 90%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 6px;
        }

        .counters {
            display: flex;
            justify-content: space-around;
            margin-bottom: 25px;
            font-weight: bold;
            font-size: 14px;
        }
        .counter-box {
            padding: 10px 15px;
            border-radius: 6px;
            color: white;
        }
        /* Green for available, Red for occupied */
        #available-counter { background-color: #28a745; }
        #occupied-counter { background-color: #dc3545; }

        .parking-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            margin-bottom: 20px;
        }

        .slot {
            padding: 20px;
            border-radius: 8px;
            color: white;
            font-weight: bold;
            transition: all 0.3s ease;
        }
        .available { background-color: #28a745; }
        .occupied { background-color: #dc3545; }

        button {
            padding: 10px 20px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 10px;
            width: 100%;
        }
        .btn-login { background-color: #007bff; color: white; }
        .btn-book { background-color: white; color: #28a745; }
        .btn-leave { background-color: white; color: #dc3545; }
        button:hover { opacity: 0.9; }

        .hidden { display: none; }
    </style>
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
</head>
<body>

    <div id="login-screen" class="container">
        <h1>Smart Parking System</h1>
        <h3>Driver Login</h3>
        <input type="text" id="username" placeholder="Username (Type anything)" required>
        <input type="password" id="password" placeholder="Password (Type anything)" required>
        <button class="btn-login" onclick="handleLogin()">Login</button>
    </div>

    <div id="dashboard-screen" class="container hidden">
        <h1>Smart Parking System</h1>
        
        <div class="counters">
            <div class="counter-box" id="available-counter">Available Slots: <span id="avail-count">2</span></div>
            <div class="counter-box" id="occupied-counter">Occupied Slots: <span id="occ-count">2</span></div>
        </div>

        <div class="parking-grid">
            <div id="slot-1" class="slot occupied">
                <div>Slot 1</div>
                <div class="status-text">Slot Occupied</div>
                <button class="btn-leave" onclick="leaveSlot(1)">Leave</button>
            </div>
            <div id="slot-2" class="slot occupied">
                <div>Slot 2</div>
                <div class="status-text">Slot Occupied</div>
                <button class="btn-leave" onclick="leaveSlot(2)">Leave</button>
            </div>
            <div id="slot-3" class="slot available">
                <div>Slot 3</div>
                <div class="status-text">Available</div>
                <button class="btn-book" onclick="bookSlot(3)">Book Now</button>
            </div>
            <div id="slot-4" class="slot available">
                <div>Slot 4</div>
                <div class="status-text">Available</div>
                <button class="btn-book" onclick="bookSlot(4)">Book Now</button>
            </div>
        </div>
    </div>

    <script>
        // Firebase Database Link
        const firebaseConfig = { 
            databaseURL: "https://smart-e9b33-default-rtdb.firebaseio.com/" 
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        // Variables start at 2
        let availableCount = 2;
        let occupiedCount = 2;

        function handleLogin() {
            const user = document.getElementById('username').value;
            const pass = document.getElementById('password').value;
            
            if(user.trim() !== "" && pass.trim() !== "") {
                document.getElementById('login-screen').classList.add('hidden');
                document.getElementById('dashboard-screen').classList.remove('hidden');
                
                // Set default Firebase database state when logged in
                db.ref('Parking/Slot1').set('Occupied');
                db.ref('Parking/Slot2').set('Occupied');
                db.ref('Parking/Slot3').set('Available');
                db.ref('Parking/Slot4').set('Available');
            } else {
                alert("Please enter a username and password.");
            }
        }

        function bookSlot(slotId) {
            const confirmBooking = confirm("Are you sure you want to book Slot " + slotId + "?");
            
            if (confirmBooking) {
                const slotDiv = document.getElementById(`slot-${slotId}`);
                
                slotDiv.className = "slot occupied";
                slotDiv.querySelector('.status-text').innerText = "Slot Occupied";
                slotDiv.querySelector('button').className = "btn-leave";
                slotDiv.querySelector('button').innerText = "Leave";
                slotDiv.querySelector('button').setAttribute("onclick", `leaveSlot(${slotId})`);
                
                availableCount--;
                occupiedCount++;
                updateCounters();

                db.ref('Parking/Slot' + slotId).set('Occupied');
            }
        }

        function leaveSlot(slotId) {
            alert("Please pay ₹100 at the counter.\nThanks for visiting!!");
            
            const slotDiv = document.getElementById(`slot-${slotId}`);
            
            slotDiv.className = "slot available";
            slotDiv.querySelector('.status-text').innerText = "Available";
            slotDiv.querySelector('button').className = "btn-book";
            slotDiv.querySelector('button').innerText = "Book Now";
            slotDiv.querySelector('button').setAttribute("onclick", `bookSlot(${slotId})`);
            
            availableCount++;
            occupiedCount--;
            updateCounters();

            db.ref('Parking/Slot' + slotId).set('Available');
        }

        function updateCounters() {
            document.getElementById('avail-count').innerText = availableCount;
            document.getElementById('occ-count').innerText = occupiedCount;
        }
    </script>
</body>
</html>
