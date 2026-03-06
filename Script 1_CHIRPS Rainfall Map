// SahelCast — CHIRPS Rainfall Map for Ouaddaï, Chad
// Planting season 2023 (June–September)

var ouaddai = ee.Geometry.Rectangle([20.0, 11.5, 24.5, 16.0]);

var chirps_collection = ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
  .filterDate('2023-06-01', '2023-09-30')
  .filterBounds(ouaddai)
  .select('precipitation');

var total_rainfall = chirps_collection.sum();

// Fix: set map center manually using coordinates instead of centerObject
Map.setCenter(22.0, 13.5, 7);

Map.addLayer(total_rainfall, {
  min: 0, max: 600,
  palette: ['red', 'orange', 'yellow', 'green', 'blue']
}, 'Total Rainfall June-Sept 2023');

print('Image count:', chirps_collection.size());
print('Ouaddaï seasonal rainfall loaded ✅', total_rainfall);
