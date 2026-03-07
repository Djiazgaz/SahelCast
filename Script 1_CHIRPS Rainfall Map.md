// SahelCast Script 1 — CHIRPS Seasonal Rainfall + Flood Detection
// Ouaddaï Province | Updated: added MODIS water mask

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);
var centerpt = ee.Geometry.Point([22.0, 13.5]); // Abéché zone

// --- LAYER 1: CHIRPS Seasonal Rainfall (June–Sept current year) ---
var SEASONYEAR = '2023';

var seasonalRain = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate(SEASONYEAR + '-06-01', SEASONYEAR + '-09-30')
  .filterBounds(ouaddai)
  .select('precipitation')
  .sum()
  .rename('seasonal_rainfall');

// --- LAYER 2: 20-Year Seasonal Baseline (1981–2020) ---
var years = ee.List.sequence(1981, 2020);
var annualTotals = years.map(function(y) {
  var yStr = ee.Number(y).format('%d');
  return ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
    .filterDate(ee.Date(yStr.cat('-06-01')), ee.Date(yStr.cat('-09-30')))
    .filterBounds(ouaddai)
    .select('precipitation')
    .sum();
});
var baseline = ee.ImageCollection(annualTotals).mean().rename('baseline_rainfall');

// --- LAYER 3 (NEW): MODIS Flood / Standing Water Detection ---
// NEW: Detects waterlogged or flood-risk areas — critical for Ouaddaï's
// wadi systems and refugee camp zones (Farchana)
var today = SEASONYEAR + '-09-30';
var fifteenago = SEASONYEAR + '-09-15';
var waterMask = ee.ImageCollection('MODIS/006/MOD44W')
  .filterDate(fifteenago, today)
  .select('water_mask')
  .mean()
  .rename('flood_standing_water');

// --- DISPLAY ---
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(
  seasonalRain.clip(ouaddai),
  {min: 0, max: 800, palette: ['red','orange','yellow','green','blue']},
  'CHIRPS Seasonal Rainfall ' + SEASONYEAR
);
Map.addLayer(
  baseline.clip(ouaddai),
  {min: 0, max: 800, palette: ['red','orange','yellow','green','blue']},
  '20-Year Rainfall Baseline 1981-2020'
);
Map.addLayer(
  waterMask.clip(ouaddai),
  {min: 0, max: 1, palette: ['white', 'blue']},
  'Flood / Standing Water (NEW)' // NEW layer
);

print('Seasonal rainfall loaded', seasonalRain);
print('Baseline loaded', baseline);
print('Flood mask loaded (NEW)', waterMask);
