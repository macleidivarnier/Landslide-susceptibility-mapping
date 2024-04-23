# Landslide-susceptibility-mapping
In this script is described a code for mapping susceptibility to landslides using morphometric indices and the Google Earth Engine platform with Java Script language
// Landslide susceptibility mapping
// Author: Macleidi Varnier
// Geographer and Master's student in Remote Sensing - UFRGS
// Date: 22/04/2024

//Due to the momentary unavailability of the TAGEE package, the variables 
//derived from this application have been deleted from this script.

/*************************Define study area**********************************/
var roi = sp_mun
Map.addLayer(roi, null, 'roi')
Map.addLayer(landslide_scars, null, 'Land_slide_scar')

//Zoom to study area
Map.centerObject(roi, 11)

/***********************Database composition ******************************/

//Elevation from SRTM
var dataset = ee.Image('NASA/NASADEM_HGT/001').clip(roi);
var elevation = dataset.select('elevation');
var elevation = elevation.reproject('EPSG:4326', null, 30)
//Map.addLayer(elevation, null, 'Elevation')

//Slope from SRTM
var slope = ee.Terrain.slope(elevation);
var slope = slope.reproject('EPSG:4326', null, 30)
//Map.addLayer(slope, null, 'Slope')

//Aspect from SRTM
var aspect = ee.Terrain.aspect(elevation);
var aspect = aspect.reproject('EPSG:4326', null, 30)
//Map.addLayer(aspect,null, 'Aspect')

//HillShade from SRTM
var hillshade = ee.Terrain.hillshade(elevation, 90, 45)
var hillshade = hillshade.reproject('EPSG:4326', null, 30)
//Map.addLayer(hillshade, null, 'Hilshade')

//Plannar curvature

// Importing module
//var TAGEE = require('users/joselucassafanelli/TAGEE:TAGEE-functions');

// Smoothing filter.
//var gaussianFilter = ee.Kernel.gaussian({
//  radius: 3, sigma: 2, units: 'pixels', normalize: true
//});

//Extend elevation from SRTM
var dataset_extend = ee.Image('NASA/NASADEM_HGT/001').clip(roi_extend);
var elevation_extend = dataset_extend.select('elevation');

// Smoothing the DEM with the gaussian kernel.
//var demSRTM = elevation_extend.convolve(gaussianFilter).resample("bilinear");

// Terrain analysis
//var DEMAttributes = TAGEE.terrainAnalysis(TAGEE, demSRTM, roi_extend);
//print(DEMAttributes.bandNames(), 'Parameters of Terrain');

//var plannarcurvature = DEMAttributes.select(6).clip(roi)
//var plannarcurvature = plannarcurvature.reproject('EPSG:4326', null, 30)
//Map.addLayer(plannarcurvature, null, 'Plannar Curvature');

//Profile Curvature
//var profilecurvature = DEMAttributes.select(7).clip(roi)
//var profilecurvature = profilecurvature.reproject('EPSG:4326', null, 30)
//Map.addLayer(profilecurvature, null, 'Profile Curvature');

//Northness
//var Northness = DEMAttributes.select(4).clip(roi)
//var Northness = Northness.reproject('EPSG:4326', null, 30)
//Map.addLayer(Northness, null, 'Northless');

//Eastness
//var Eastness = DEMAttributes.select(5).clip(roi)
//var Eastness = Eastness.reproject('EPSG:4326', null, 30)
//Map.addLayer(Eastness, null, 'Eastness');

//GaussianCurvature
//var GaussianCurvature = DEMAttributes.select(9).clip(roi)
//var GaussianCurvature = GaussianCurvature.reproject('EPSG:4326', null, 30)
//Map.addLayer(GaussianCurvature, null, 'Guassian Curvature');

////Shape index
//var shapeindex = DEMAttributes.select(12).clip(roi)
//var shapeindex = shapeindex.reproject('EPSG:4326', null, 30)
//Map.addLayer(shapeindex, null, 'shapeindex');

//Flow Accumulation
var flowaccumulation = ee.Image("MERIT/Hydro/v1_0_1").select('upa').log().clip(roi)
var flowaccumulation_ = ee.Image("MERIT/Hydro/v1_0_1").select('upa').clip(roi)
//var flowaccumulation = ee.ImageCollection('users/imerg/flow_acc_3s').mean().log().clip(roi)
var flowaccumulation = flowaccumulation.reproject('EPSG:4326', null, 30).rename('FlowAcc')
Map.addLayer(flowaccumulation, null, 'Flow Accumulation')

//HAND
var hand = ee.Image("users/gena/GlobalHAND/30m/hand-1000").clip(roi).rename('HAND')
var hand = hand.reproject('EPSG:4326', null, 30)
//Map.addLayer(hand, null, 'HAND')

//TPI
var meanTPI = elevation_extend.focalMean(5, 'square');
var tpi = elevation_extend.subtract(meanTPI).reproject('EPSG:4326', null , 30).rename('tpi').clip(roi);
Map.addLayer(tpi, null, 'TPI');

//horizontal distance to channel network
var rivers = ee.Image("MERIT/Hydro/v1_0_1").select('upa').clip(roi).gt(0.5)
Map.addLayer(rivers)

// Euclidean Distance.
var maxDistM = 7500;  // 10 km

// Calculate distance to target pixels.
var euclideanKernel = ee.Kernel.euclidean(maxDistM, 'meters');
var visParamsEuclideanDist = {min: 0, max: maxDistM};

var hdtp = rivers.distance(euclideanKernel);
var hdtp = hdtp.reproject('EPSG:4326', null, 30).rename('hdtp')
//Map.addLayer(hdtp, visParamsEuclideanDist, 'HDTP');

//ls Factor
var ls_factors = ee.Image([flowaccumulation, slope]).rename(['flowaccumulation_','slope'])
var factorLS = ls_factors.expression(
 '(FACC*270/22.13)**0.4*(SLOPE/0.0896)**1.3', 
 { 
 'FACC': ls_factors.select('flowaccumulation_'), 
 'SLOPE': ls_factors.select('slope') });

var factorLS = factorLS.reproject('EPSG:4326', null, 30)
//Map.addLayer(factorLS, null, 'LSfactor')

// Composite a cube data.
var cubedata = slope.addBands(aspect)
                    .addBands(hillshade)
//                    .addBands(plannarcurvature)
//                    .addBands(profilecurvature)
//                    .addBands(Northness)
//                    .addBands(Eastness)
//                    .addBands(GaussianCurvature)
                    .addBands(hand)
                    //.addBands(twi)
                    .addBands(flowaccumulation)
                    .addBands(tpi)
                    .addBands(hdtp)
                    //.addBands(factorLS)
                    //.addBands(MaximalCurvature)
                    //.addBands(MinimalCurvature)
                    //.addBands(ShapeIndex)

/*****************Creation of landslide occurrence samples*************************/
//Map.addLayer(landslide_scars, null, 'Landslide Scars')
var occurence = ee.FeatureCollection.randomPoints(landslide_scars, 5000)
    .map(function(f) {
        return f.set('class', 1)
    })

//Creating a 60m buffer around the scars
var buffersize40 =  60;

var create40mBuffer = function(feature) {
  return feature.buffer(buffersize40);
};

var buffer40 = landslide_scars.map(create40mBuffer)

var buffersize250 =  5000;

var create250mBuffer = function(feature) {
  return feature.buffer(buffersize250);
};

var buffer250 = landslide_scars.map(create250mBuffer)

// Creation of landslide non-occurrence samples
var non_occurence = ee.FeatureCollection.randomPoints(roi, 5007)
    .map(function(f) {
        return f.set('class', 0)
    })

var non_occurence = non_occurence.filter(ee.Filter.bounds(buffer40).not())
//print(non_occurence)

// Group training data
var traininggroup = occurence.merge(non_occurence)

// Extract the values from the input data for each point training
var trainingvalues = cubedata.reduceRegions(traininggroup, ee.Reducer.mean());

// Define the band Names
var bandNames = cubedata.bandNames();
print(bandNames, "band Names")

//Filter out null values from the training feature collection
var trainingDataset = trainingvalues.randomColumn('random');
var trainingNoNulls = trainingDataset.filter(
  ee.Filter.notNull(trainingDataset.first().propertyNames())
);

// Sample the training points data
var training = trainingNoNulls.filter(ee.Filter.lte('random', 0.7));
var validation = trainingNoNulls.filter(ee.Filter.gt('random', 0.7));

print(validation)

Export.table.toDrive({
collection: validation,
  description: 'Validation_samples',
fileFormat: 'CSV'
});

/********************************Box Plot *************************************/
//statistical evaluation of occurrence and non-occurrence samples
// generate input for box plot per class
var Names = ee.List(['0','1'])

function showBoxPlot(validation, bands) {
  var dataTable = {
    cols: [
      {id: 'x', type: 'string'},
      {id: 'series0', type: 'number'}, // dummy series
      {id: 'min', type: 'number', role: 'interval'},
      {id: 'max', type: 'number', role: 'interval'},
      {id: 'firstQuartile', type: 'number', role: 'interval'},
      {id: 'median', type: 'number', role: 'interval'},
      {id: 'thirdQuartile', type:'number', role: 'interval'}
    ]
  }
  
  var values = Names.map(function(c) {
    var index = Names.indexOf(c)
    var v = validation.filter(ee.Filter.eq('class', index)).aggregate_array(bands)
    var min = v.reduce(ee.Reducer.min())
    var max = v.reduce(ee.Reducer.max())
    var p25 = v.reduce(ee.Reducer.percentile([25]))
    var p50 = v.reduce(ee.Reducer.percentile([50]))
    var p75 = v.reduce(ee.Reducer.percentile([75]))
  
    return [c, 0, min, max, p25, p50, p75]
  })
  
  values.evaluate(function(values) {
    dataTable.rows = values.map(function(row) {
      return { c: row.map(function(o) { return { v: o } }) }
    })
    
    var options = {
      title:'Box Plot ' + bands,
      height: 500,
      legend: {position: 'none'},
      hAxis: {
        gridlines: {color: '#fff'}
      },
      lineWidth: 0,
      series: [{'color': '#D3362D'}],
      intervals: {
        barWidth: 1,
        boxWidth: 1,
        lineWidth: 2,
        style: 'boxes'
      },
      interval: {
        min: {
          style: 'bars',
          fillOpacity: 1,
          color: '#777'
        },
        max: {
          style: 'bars',
          fillOpacity: 1,
          color: '#777'
        }
      }
    };
    print(ui.Chart(dataTable, 'LineChart', options));
  })
}

showBoxPlot(validation, 'tpi')

/********************************Classifier*************************************/
// Random Forest
var rf = ee.Classifier.smileRandomForest(128).train(training, 'class', bandNames).setOutputMode('PROBABILITY');

//probability mapping
var rfclass = cubedata.select(bandNames).classify(rf);//.updateMask(naturalvegetation);

// Visualization parameter
var viz = {min:0, max:1, palette:['green','yellow','red']};    
Map.addLayer(rfclass, viz,'RF');

//Important factor of variables to the classification
// RF Classifier 
var rf_dict = rf.explain();
print('Explain:',rf_dict);
 
var rf_variable_importance = ee.Feature(null, ee.Dictionary(rf_dict).get('importance'));
 
var rf_chart =
ui.Chart.feature.byProperty(rf_variable_importance)
.setChartType('ColumnChart')
.setOptions({
title: 'Random Forest Variable Importance',
legend: {position: 'none'},
hAxis: {title: 'Bands'},
vAxis: {title: 'Importance'}
});
 
print(rf_chart);

/*****************************************Acuracy Evaluation**************************************/
// Calculate the Receiver Operating Characteristic (ROC) curve
// Random Forest Classifier AUC

// Chance these as needed
var FF = ee.FeatureCollection(validation).filterMetadata('class','equals',1)
var NFF = ee.FeatureCollection(validation).filterMetadata('class','equals',0)
var FFrf = rfclass.reduceRegions(FF,ee.Reducer.max().setOutputs(['class']),30).map(function(x){return x.set('is_target',1);})
var NFFrf = rfclass.reduceRegions(NFF,ee.Reducer.max().setOutputs(['class']),30).map(function(x){return x.set('is_target',0);})
var combined = FFrf.merge(NFFrf)
//print(combined,'combine')

// Show NDVI of points
//print(FFrf.aggregate_array('class'),'Change')
//print(NFFrf.aggregate_array('class'),'Nochange')

//chance ROC curve
var ROC_field = 'class', ROC_min = 0, ROC_max = 1, ROC_steps = 200, ROC_points = combined

var ROC = ee.FeatureCollection(ee.List.sequence(ROC_min, ROC_max, null, ROC_steps).map(function (cutoff) {
  var target_roc = ROC_points.filterMetadata('is_target','equals',1)
  // true-positive-rate, sensitivity  
  var TPR = ee.Number(target_roc.filterMetadata(ROC_field,'greater_than',cutoff).size()).divide(target_roc.size()) 
  var non_target_roc = ROC_points.filterMetadata('is_target','equals',0)
  // true-negative-rate, specificity  
  var TNR = ee.Number(non_target_roc.filterMetadata(ROC_field,'less_than',cutoff).size()).divide(non_target_roc.size()) 
  return ee.Feature(null,{cutoff: cutoff, TPR: TPR, TNR: TNR, FPR:TNR.subtract(1).multiply(-1),  dist:TPR.subtract(1).pow(2).add(TNR.subtract(1).pow(2)).sqrt()})
}))
// Use trapezoidal approximation for area under curve (AUC)
var X = ee.Array(ROC.aggregate_array('FPR')), 
    Y = ee.Array(ROC.aggregate_array('TPR')), 
    Xk_m_Xkm1 = X.slice(0,1).subtract(X.slice(0,0,-1)),
    Yk_p_Ykm1 = Y.slice(0,1).add(Y.slice(0,0,-1)),
    AUC = Xk_m_Xkm1.multiply(Yk_p_Ykm1).multiply(0.5).reduce('sum',[0]).abs().toList().get(0)
print(AUC,'Area under curve')
// Plot the ROC curve
print(ui.Chart.feature.byFeature(ROC, 'FPR', 'TPR').setOptions({
      title: 'ROC curve',
      legend: 'none',
      hAxis: { title: 'False-positive-rate'},
      vAxis: { title: 'True-negative-rate'},
      lineWidth: 1}))
// find the cutoff value whose ROC point is closest to (0,1) (= "perfect classification")      
var ROC_best = ROC.sort('dist').first().get('cutoff').aside(print,'best ROC point cutoff')
