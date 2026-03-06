// SahelCast — Script 5: Planting Advisory — June 2023 VALIDATION 
// Expected output: GREEN or YELLOW (peak planting season)

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

var today       = '2023-06-15';
var fifteen_ago = '2023-06-01';  // Wider 30-day window for sparse SMAP coverage

// ── LAYER 1: CHIRPS Rainfall ──────────────────────────────────
var rain_safe = ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
  .filterDate(fifteen_ago, today)
  .filterBounds(ouaddai)
  .select('precipitation')
  .merge(ee.ImageCollection([ee.Image(0).rename('precipitation')]))
  .sum()
  .unmask(0)
  .rename('recent_rain');

// ── LAYER 2: SMAP Soil Moisture ───────────────────────────────
var smap_safe = ee.ImageCollection("NASA/SMAP/SPL3SMP_E/006")
  .filterDate('2023-06-01', today)  // Full season window for SMAP
  .filterBounds(ouaddai)
  .select('soil_moisture_am')
  .merge(ee.ImageCollection([ee.Image(0).rename('soil_moisture_am')]))
  .mean()
  .unmask(0)
  .rename('soil_moisture');

// ── LAYER 3: NDVI Anomaly ─────────────────────────────────────
var ndvi_current = ee.ImageCollection("MODIS/061/MOD13Q1")
  .filterDate('2023-06-01', today)  // Full season window for MODIS
  .filterBounds(ouaddai)
  .select('NDVI')
  .merge(ee.ImageCollection([ee.Image(0).rename('NDVI')]))
  .mean()
  .multiply(0.0001)
  .unmask(0);

var ndvi_baseline = ee.ImageCollection("MODIS/061/MOD13Q1")
  .filterDate('2000-06-01', '2020-09-30')
  .filterBounds(ouaddai)
  .select('NDVI')
  .mean()
  .multiply(0.0001)
  .unmask(0);

var ndvi_safe = ndvi_current.subtract(ndvi_baseline).unmask(0);

// ── ADVISORY LOGIC ────────────────────────────────────────────
var green_condition = rain_safe.gte(25)
  .and(smap_safe.gte(0.15))
  .and(ndvi_safe.gte(-0.05));

var yellow_condition = rain_safe.gte(15)
  .and(smap_safe.gte(0.10))
  .and(green_condition.not());

var advisory = ee.Image(0)
  .where(yellow_condition, 1)
  .where(green_condition, 2)
  .rename('planting_advisory');

// ── DISPLAY ───────────────────────────────────────────────────
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(rain_safe.clip(ouaddai), {
  min: 0, max: 300,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
}, '30-Day Rainfall (Jul-Aug 2023)');

Map.addLayer(smap_safe.clip(ouaddai), {
  min: 0.0, max: 0.4,
  palette: ['red', 'orange', 'yellow', 'lightblue', 'blue']
}, 'Soil Moisture (Jun-Aug 2023)');

Map.addLayer(advisory.clip(ouaddai), {
  min: 0, max: 2,
  palette: ['red', 'yellow', 'green']
}, '🌱 SahelCast Advisory - Aug 2023');

// ── SCORE ─────────────────────────────────────────────────────
var center_point = ee.Geometry.Point([22.0, 13.5]);

advisory.sample({
  region: center_point,
  scale: 5000,
  numPixels: 1,
  geometries: false
}).first().get('planting_advisory')
.evaluate(function(val, err) {
  if (err) { print('Error:', err); return; }

  var s = (val !== null && val !== undefined) ? val : 0;

  print('─────────────────────────────────────');
  print('🌱 SahelCast VALIDATION — June 15, 2023');
  print('Advisory score:', s);
  print('  0 = 🔴 RED    → Do Not Plant');
  print('  1 = 🟡 YELLOW → Wait 7 Days');
  print('  2 = 🟢 GREEN  → Plant Now');

  if      (s < 0.5) print('Status: 🔴 DO NOT PLANT');
  else if (s < 1.5) print('Status: 🟡 WAIT — Monitor Conditions');
  else              print('Status: 🟢 PLANT NOW — Conditions Favorable');

  print('Expected: 🟢 GREEN or 🟡 YELLOW (Peak Season)');
  print('─────────────────────────────────────');
});
