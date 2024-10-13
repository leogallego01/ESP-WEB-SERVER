<html>
<head>
  <title>ESP Web Server</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" href="data:,">
  <style>
    html {font-family: Arial; display: inline-block; text-align: center;}
    h2 {font-size: 3.0rem;}
    p {font-size: 3.0rem;}
    body {max-width: 600px; margin:0px auto; padding-bottom: 25px;}
    .switch {position: relative; display: inline-block; width: 120px; height: 68px}
    .switch input {display: none}
    .slider {position: absolute; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; border-radius: 6px}
    .slider:before {position: absolute; content: ""; height: 52px; width: 52px; left: 8px; bottom: 8px; background-color: #fff; -webkit-transition: .4s; transition: .4s; border-radius: 3px}
    input:checked+.slider {background-color: #b30000}
    input:checked+.slider:before {-webkit-transform: translateX(52px); -ms-transform: translateX(52px); transform: translateX(52px)}
  </style>
</head>
<body>
  <h2>ESP Web Server</h2>
  
  <h4>Output - GPIO 2</h4>
  <label class="switch">
    <input type="checkbox" id="gpio2" onchange="toggleGPIO(2)">
    <span class="slider"></span>
  </label>
  
  <h4>Output - GPIO 4</h4>
  <label class="switch">
    <input type="checkbox" id="gpio4" onchange="toggleGPIO(4)">
    <span class="slider"></span>
  </label>

  <h4>Output - GPIO 33</h4>
  <label class="switch">
    <input type="checkbox" id="gpio33" onchange="toggleGPIO(33)">
    <span class="slider"></span>
  </label>

  <script>
    // Function to toggle GPIO pins and send the request to the server
    function toggleGPIO(pin) {
      var element = document.getElementById("gpio" + pin);
      var state = element.checked ? 1 : 0;
      
      // Save the state in localStorage
      localStorage.setItem("gpio" + pin, state);

      var xhr = new XMLHttpRequest();
      xhr.open("GET", `http://<ESP_IP>/update?output=${pin}&state=${state}`, true);
      xhr.send();

      // Reset to OFF after 1 minute (60000 ms)
      setTimeout(function() {
        resetSwitch(pin);
      }, 60000);
    }

    // Function to reset the switch to OFF
    function resetSwitch(pin) {
      document.getElementById("gpio" + pin).checked = false;
      localStorage.setItem("gpio" + pin, 0);  // Update localStorage

      var xhr = new XMLHttpRequest();
      xhr.open("GET", `http://<ESP_IP>/update?output=${pin}&state=0`, true);
      xhr.send();
    }

    // Load the saved switch states from localStorage on page load
    function loadSwitchStates() {
      ["2", "4", "33"].forEach(function(pin) {
        var savedState = localStorage.getItem("gpio" + pin);
        if (savedState === "1") {
          document.getElementById("gpio" + pin).checked = true;
        } else {
          document.getElementById("gpio" + pin).checked = false;
        }
      });
    }

    window.onload = function() {
      loadSwitchStates();  // Restore switch states on page load
    };
  </script>
</body>
</html>
