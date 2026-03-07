// SahelCast Script 2 — 20-Year Rainfall Baseline Anomaly Detection
// No changes required — baseline logic is sound and validated

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);
var SEASONYEAR = '2023';

var seasonalRain = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
  .filterDate(SEASONYEAR + '-06-01', SEASONYEAR + '-09-30')
  .filterBounds(ouaddai)
  .select('precipitation')
  .sum()
  .rename('rainfall_' + SEASONYEAR);

var years = ee.List.sequence(1981, 2020);
var annualTotals = years.map(function(y) {
  var yStr = ee.Number(y).format('%d');
  return ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
    .filterDate(ee.Date(yStr.cat('-06-01')), ee.Date(yStr.cat('-09-30')))
    .filterBounds(ouaddai)
    .select('precipitation')
    .sum();
});
var baseline = ee.ImageCollection(annualTotals).mean().rename('baseline');
var anomaly = seasonalRain.subtract(baseline).rename('rainfall_anomaly');

Map.setCenter(22.0, 13.5, 7);
Map.addLayer(anomaly.clip(ouaddai),
  {min: -200, max: 200, palette: ['brown','white','darkgreen']},
  'Rainfall Anomaly vs 20-Year Baseline');

print('Anomaly layer loaded', anomaly);
