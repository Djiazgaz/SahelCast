// SahelCast — Script 3: MODIS NDVI Vegetation Stress for Ouaddaï
// Detects crop/vegetation health during 2023 growing season

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

// Load MODIS NDVI - 16-day composite at 250m
var modis_ndvi = ee.ImageCollection("MODIS/061/MOD13Q1")
  .filterDate('2023-06-01', '2023-09-30')
  .filterBounds(ouaddai)
  .select('NDVI');

// MODIS NDVI is scaled by 0.0001 — apply scale factor
var ndvi_scaled = modis_ndvi.map(function(img) {
  return img.multiply(0.0001).copyProperties(img, ['system:time_start']);
});

// Mean NDVI for the 2023 growing season
var ndvi_2023 = ndvi_scaled.mean().rename('ndvi_2023');

// Load 20-year NDVI baseline (2000-2020)
var ndvi_baseline = ee.ImageCollection("MODIS/061/MOD13Q1")
  .filterDate('2000-06-01', '2020-09-30')
  .filterBounds(ouaddai)
  .select('NDVI')
  .map(function(img){
    return img.multiply(0.0001).copyProperties(img, ['system:time_start']);
  })
  .mean()
  .rename('ndvi_baseline');

// NDVI Anomaly: 2023 vs 20-year baseline
var ndvi_anomaly = ndvi_2023.subtract(ndvi_baseline).rename('ndvi_anomaly');

// Display layers
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(ndvi_2023, {
  min: 0, max: 0.6,
  palette: ['white', 'yellow', 'green', 'darkgreen']
}, 'NDVI 2023 Growing Season');

Map.addLayer(ndvi_baseline, {
  min: 0, max: 0.6,
  palette: ['white', 'yellow', 'green', 'darkgreen']
}, 'NDVI 20-Year Baseline');

Map.addLayer(ndvi_anomaly, {
  min: -0.2, max: 0.2,
  palette: ['brown', 'white', 'darkgreen']
}, 'NDVI Anomaly 2023 vs Baseline');

print('NDVI 2023 loaded ✅', ndvi_2023);
print('NDVI baseline loaded ✅', ndvi_baseline);
print('NDVI anomaly loaded ✅', ndvi_anomaly);
