<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Strompreise mit Highcharts</title>
  <script src="https://code.highcharts.com/highcharts.js"></script>
  <script src="https://code.highcharts.com/modules/accessibility.js"></script>
  <style>
    body { font-family: "Segoe UI", sans-serif; margin: 10px; background: #f3f4f6; }
    h1 { text-align: center; font-size: 1.4em; margin-bottom: 10px; }
    #stats { display: flex; flex-wrap: wrap; gap: 10px; margin: 15px 0; }
    .card { background: white; padding: 12px; border-radius: 10px; flex: 1 1 100%;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1); text-align: center; }
    .card h2 { margin: 0; font-size: 1.2em; }
    .card p { margin: 5px 0 0; color: #555; font-size: 0.9em; }
    @media (min-width: 600px) { .card { flex: 1 1 calc(33% - 10px); } }
    #container { height: 400px; background: white; border-radius: 10px; }
    #controls { text-align: center; margin: 10px 0; }
    #controls input, #controls button { padding: 6px 10px; margin: 5px; }
  </style>
</head>
<body>
  <div id="controls">
    <input type="date" id="dateInput">
    <button onclick="loadData()">🔄 Laden</button>
  </div>

  <div id="stats">
    <div class="card"><h2 id="min"></h2><p>Tiefstpreis</p></div>
    <div class="card"><h2 id="max"></h2><p>Höchstpreis</p></div>
    <div class="card"><h2 id="avg"></h2><p>Durchschnitt</p></div>
  </div>

  <div id="container"></div>

  <script>
    // Hilfsfunktion: ISO mit korrektem Offset
    function toIsoWithOffset(date) {
      const tzo = -date.getTimezoneOffset();
      const dif = tzo >= 0 ? "+" : "-";
      const pad = n => String(Math.floor(Math.abs(n))).padStart(2, "0");
      return date.getFullYear() +
        "-" + pad(date.getMonth()+1) +
        "-" + pad(date.getDate()) +
        "T" + pad(date.getHours()) +
        ":" + pad(date.getMinutes()) +
        ":" + pad(date.getSeconds()) +
        dif + pad(tzo/60) +
        ":" + pad(tzo%60);
    }

    // Korrekte Zeitspanne Mitternacht–Mitternacht in Europe/Zurich
    function getZurichDayRange(dateStr) {
      const date = new Date(dateStr);
      const tz = "Europe/Zurich";

      const dateFormatter = new Intl.DateTimeFormat("en-CA", {
        timeZone: tz,
        year: "numeric",
        month: "2-digit",
        day: "2-digit"
      });

      const formatted = dateFormatter.format(date); // yyyy-mm-dd
      const start = new Date(formatted + "T00:00:00");
      const end = new Date(start.getTime() + 24 * 60 * 60 * 1000 - 1000);
      return { start, end };
    }

    async function loadData() {
      const input = document.getElementById("dateInput").value;
      if (!input) return;

      const { start, end } = getZurichDayRange(input);
      const startIso = toIsoWithOffset(start);
      const endIso   = toIsoWithOffset(end);

      const url = `https://api.tariffs.ekz.ch/v1/tariffs?tariff_type=integrated&tariff_name=integrated_400D_E&start_timestamp=${encodeURIComponent(startIso)}&end_timestamp=${encodeURIComponent(endIso)}`;

      try {
        const resp = await fetch(url, { headers: { accept: "application/json" }});
        const data = await resp.json();
        const values = data.prices || [];

        const times = values.map(v => new Date(v.start_timestamp).getTime());
        const prices = values.map(v => {
          const kwh = v.integrated.find(x => x.unit === "CHF/kWh");
          return kwh ? kwh.value : null;
        });

        if (prices.length === 0) {
          alert("Keine Daten für dieses Datum gefunden.");
          return;
        }

        const minVal = Math.min(...prices);
        const maxVal = Math.max(...prices);
        const avgVal = prices.reduce((a,b) => a+b, 0) / prices.length;

        const minIdx = prices.indexOf(minVal);
        const maxIdx = prices.indexOf(maxVal);

        document.getElementById("min").textContent = `${minVal.toFixed(5)} CHF/kWh (${new Date(times[minIdx]).toLocaleTimeString([], {hour:"2-digit",minute:"2-digit"})})`;
        document.getElementById("max").textContent = `${maxVal.toFixed(5)} CHF/kWh (${new Date(times[maxIdx]).toLocaleTimeString([], {hour:"2-digit",minute:"2-digit"})})`;
        document.getElementById("avg").textContent = `${avgVal.toFixed(5)} CHF/kWh`;

        Highcharts.chart('container', {
          chart: { zoomType: 'x', backgroundColor: 'white' },
          title: { text: `Tarifpreise für ${input}` },
          xAxis: { type: 'datetime', title: { text: 'Zeit' } },
          yAxis: { title: { text: 'Preis (CHF/kWh)' } },
          tooltip: { xDateFormat: '%H:%M', valueDecimals: 5 },
          series: [{
            name: "Preis",
            data: times.map((t,i)=>[t, prices[i]]),
            color: '#007bff',
            lineWidth: 2,
            marker: { enabled: false }
          }],
          credits: { enabled: false }
        });
      } catch (err) {
        console.error("Fehler:", err);
        alert("Fehler beim Laden der Daten!");
      }
    }

    // Standard: Heute laden
    window.onload = () => {
      const today = new Date().toISOString().split("T")[0];
      document.getElementById("dateInput").value = today;
      loadData();
    };
  </script>
</body>
</html>
