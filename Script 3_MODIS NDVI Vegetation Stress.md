// SahelCast Script 3 — MODIS NDVI Vegetation Stress
// No changes required

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

var modisNDVI = ee.ImageCollection('MODIS/061/MOD13Q1')
  .filterDate('2023-06-01', '2023-09-30')
  .filterBounds(ouaddai)
  .select('NDVI');

var ndviScaled = modisNDVI.map(function(img) {
  return img.multiply(0.0001).copyProperties(img, ['system:time_start']);
});

var ndvi2023 = ndviScaled.mean().rename('ndvi_2023');

var ndviBaseline = ee.ImageCollection('MODIS/061/MOD13Q1')
  .filterDate('2000-06-01', '2020-09-30')
  .filterBounds(ouaddai)
  .select('NDVI')
  .map(function(img) {
    return img.multiply(0.0001).copyProperties(img, ['system:time_start']);
  })
  .mean()
  .rename('ndvi_baseline');

var ndviAnomaly = ndvi2023.subtract(ndviBaseline).rename('ndvi_anomaly');

Map.setCenter(22.0, 13.5, 7);
Map.addLayer(ndvi2023, {min:0, max:0.6, palette:['white','yellow','green','darkgreen']}, 'NDVI 2023');
Map.addLayer(ndviBaseline, {min:0, max:0.6, palette:['white','yellow','green','darkgreen']}, 'NDVI 20-Year Baseline');
Map.addLayer(ndviAnomaly, {min:-0.2, max:0.2, palette:['brown','white','darkgreen']}, 'NDVI Anomaly 2023 vs Baseline');

print('NDVI 2023 loaded', ndvi2023);
print('NDVI baseline loaded', ndviBaseline);
print('NDVI anomaly loaded', ndviAnomaly);
