// SahelCast — Script 2: 20-Year Rainfall Baseline for Ouaddaï

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

// Function to get seasonal total using string dates
function getSeasonalTotal(year) {
  var y = ee.Number(year).int();
  var start = ee.Date(y.format('%d').cat('-06-01'));
  var end   = ee.Date(y.format('%d').cat('-09-30'));
  return ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
    .filterDate(start, end)
    .filterBounds(ouaddai)
    .select('precipitation')
    .sum()
    .set('year', year);
}

// Build list of years 2000–2020
var years = ee.List.sequence(2000, 2020);

// Map over all years
var yearly_rainfall = ee.ImageCollection(
  years.map(function(y){ return getSeasonalTotal(y); })
);

// Compute 20-year mean baseline
var baseline_mean = yearly_rainfall.mean().rename('baseline_mean');

// Compute 2023 seasonal total
var season_2023 = getSeasonalTotal(2023).rename('rainfall_2023');

// Compute anomaly: 2023 vs baseline
var anomaly_2023 = season_2023.subtract(baseline_mean).rename('anomaly');

// Display all layers
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(baseline_mean, {
  min: 0, max: 600,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
}, '20-Year Baseline Mean (2000-2020)');

Map.addLayer(season_2023, {
  min: 0, max: 600,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
}, '2023 Seasonal Rainfall');

Map.addLayer(anomaly_2023, {
  min: -200, max: 200,
  palette: ['brown', 'white', 'darkblue']
}, '2023 Anomaly vs Baseline');

print('Baseline mean loaded ✅', baseline_mean);
print('2023 anomaly loaded ✅', anomaly_2023);
