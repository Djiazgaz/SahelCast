// SahelCast Script 4 — SMAP Soil Moisture + iSDA Dynamic Soil Thresholds
// FIXED: NASA/SMAP/SPL3SMP_E/005  ← underscore between SMP_E was missing

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

// ─────────────────────────────────────────────────────────────
// LAYER 1: SMAP Soil Moisture 2023 (June–Sept)
// Asset: NASA/SMAP/SPL3SMP_E/005  covers 2015-03-31 → 2023-12-03
// Band:  soil_moisture_am  (volume fraction, ~0–0.5 cm³/cm³)
// ─────────────────────────────────────────────────────────────
var smap = ee.ImageCollection('NASA/SMAP/SPL3SMP_E/005')
  .filterDate('2023-06-01', '2023-09-30')
  .filterBounds(ouaddai)
  .select('soil_moisture_am');

var smap2023 = smap.mean().rename('soil_moisture_2023');

// ─────────────────────────────────────────────────────────────
// LAYER 2: SMAP Baseline (2016–2020)
// GEE also provides pre-computed soil_moisture_am_anomaly band —
// but we keep manual baseline for transparency in methodology
// ─────────────────────────────────────────────────────────────
var smapBaseline = ee.ImageCollection('NASA/SMAP/SPL3SMP_E/005')
  .filterDate('2016-06-01', '2020-09-30')
  .filterBounds(ouaddai)
  .select('soil_moisture_am')
  .mean()
  .rename('soil_moisture_baseline');

var smapAnomaly = smap2023.subtract(smapBaseline)
  .rename('soil_moisture_anomaly');

// ─────────────────────────────────────────────────────────────
// BONUS: GEE pre-computed anomaly band (experimental, built-in)
// No baseline calculation needed — compare your manual vs this
// ─────────────────────────────────────────────────────────────
var smapBuiltInAnomaly = ee.ImageCollection('NASA/SMAP/SPL3SMP_E/005')
  .filterDate('2023-06-01', '2023-09-30')
  .filterBounds(ouaddai)
  .select('soil_moisture_am_anomaly')
  .mean()
  .rename('builtin_anomaly');

// ─────────────────────────────────────────────────────────────
// LAYER 3: iSDA Dynamic Soil Thresholds (already working ✓)
// Asset: ISDASOIL/Africa/v1/texture_class  band 0 = 0–20 cm
// USDA classes 1–12: Sandy(1–3) | Loamy(4–6) | Clay(7–12)
// ─────────────────────────────────────────────────────────────
var soilTexture = ee.Image('ISDASOIL/Africa/v1/texture_class')
  .select('texture_0_20')   // ← confirmed band name for 0–20 cm depth
  .clip(ouaddai);

var moistureThreshold = soilTexture
  .where(soilTexture.lte(3),  0.10)   // Sandy → lower threshold
  .where(soilTexture.gt(3).and(soilTexture.lte(6)), 0.15)  // Loamy → standard
  .where(soilTexture.gt(6),   0.18)   // Clay  → higher threshold
  .unmask(0.15) 
  .rename('dynamic_planting_threshold');

var smapThresholdGreen  = moistureThreshold;
var smapThresholdYellow = moistureThreshold.multiply(0.67);

// Planting trigger: 1 = sufficient moisture, 0 = too dry
var plantingTrigger = smap2023.gt(moistureThreshold)
  .rename('planting_trigger');

// ─────────────────────────────────────────────────────────────
// DISPLAY
// ─────────────────────────────────────────────────────────────
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(smap2023.clip(ouaddai),
  {min:0.0, max:0.4, palette:['red','orange','yellow','lightblue','blue']},
  'Soil Moisture 2023 (cm³/cm³)');

Map.addLayer(smapBaseline.clip(ouaddai),
  {min:0.0, max:0.4, palette:['red','orange','yellow','lightblue','blue']},
  'Soil Moisture Baseline 2016–2020');

Map.addLayer(smapAnomaly.clip(ouaddai),
  {min:-0.1, max:0.1, palette:['brown','white','darkblue']},
  'Soil Moisture Anomaly 2023 (manual)');

Map.addLayer(smapBuiltInAnomaly.clip(ouaddai),
  {min:-0.1, max:0.1, palette:['brown','white','darkblue']},
  'Soil Moisture Anomaly (GEE built-in)');

Map.addLayer(soilTexture.clip(ouaddai),
  {min:1, max:12, palette:['yellow','orange','brown','red']},
  'iSDA Soil Texture Class (0–20 cm)');

Map.addLayer(moistureThreshold.clip(ouaddai),
  {min:0.10, max:0.18, palette:['yellow','green','blue']},
  'Dynamic Planting Threshold (per soil type)');

Map.addLayer(plantingTrigger.clip(ouaddai),
  {min:0, max:1, palette:['red','green']},
  'Planting Trigger — Green=Plant / Red=Wait');

// ─────────────────────────────────────────────────────────────
// CONSOLE OUTPUT
// ─────────────────────────────────────────────────────────────
print('SMAP 2023 loaded', smap2023);
print('SMAP baseline loaded', smapBaseline);
print('iSDA soil texture loaded ✓', soilTexture);
print('Dynamic threshold loaded ✓', moistureThreshold);
print('SMAP image count:', smap.size());
print('');
print('Expected: SMAP image count = 30–60 images (June–Sept 2023)');
print('Soil Moisture 2023 range: ~0.05 (dry) to ~0.35 (wet)');
print('Dynamic threshold range: 0.10 (sandy N.Ouadda) to 0.18 (clay S.Ouadda)');

