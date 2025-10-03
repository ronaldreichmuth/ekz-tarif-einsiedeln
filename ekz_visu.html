<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="="UTF-8">
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
  </style>
</head>
<body>
  <h1>EKZ Strompreise</h1>
  <div id="stats">
    <div class="card"><h2 id="min"></h2><p>Tiefstpreis</p></div>
    <div class="card"><h2 id="max"></h2><p>Höchstpreis</p></div>
    <div class="card"><h2 id="avg"></h2><p>Durchschnitt</p></div>
  </div>
  <div id="container"></div>

  <script>
    async function loadData() {
      const today = new Date().toISOString().split("T")[0];
      const start = `${today}T00:00:00+02:00`;
      const end   = `${today}T23:59:59+02:00`;
      const url = `https://api.tariffs.ekz.ch/v1/tariffs?tariff_type=integrated&tariff_name=integrated_400D_E&start_timestamp=${encodeURIComponent(start)}&end_timestamp=${encodeURIComponent(end)}`;

      const resp = await fetch(url, { headers: { accept: "application/json" }});
      const data = await resp.json();
      const values = data.prices || [];

      const times = values.map(v => new Date(v.start_timestamp).getTime());
      const prices = values.map(v => {
        const kwh = v.integrated.find(x => x.unit === "CHF/kWh");
        return kwh ? kwh.value : null;
      });

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
        title: { text: `Tarifpreise für ${today}` },
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
    }

    loadData();
  </script>
</body>
</html>
