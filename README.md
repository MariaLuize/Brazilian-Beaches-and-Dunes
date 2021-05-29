# BRAZILIAN BEACHES AND DUNES STATUS: THREE DECADES OF DETECTION USING MACHINE LEARNING  

Brazilian beaches play an essential role in maintaining human populations and conserving biodiversity, being primarily impacted by anthropogenic and natural activities, such as tourism, pollution, and sea-level variations. The present work uses the Random Forest algorithm to automatically and systematically classify sandy coastal bodies as "beaches or dunes" from 1985 to 2019 throughout the Brazilian Coastal Zone (BCZ). In Brazil, the areas covered by beaches and dunes reduced ~714 km² (around 17% of the national cover). The South and Southeast regions experimented with more significant area reductions compared to the North and Northeast regions. Regarding concordance levels with pre-existent data, the general agreement of 1985 and 2019 data reached 0.85 and 0.97 of global agreement levels.

## Visualization Script (GEE)
Visualization of final classification and statistics: https://code.earthengine.google.com/51319edac2a0268ea71ad806e81f8e86

## Visual Results
* Base Mosaic
![](/images/cropbaseMosaic.png)

 * Classification Maps
 
Reference Map - MapBiomas 4.1 Classification             | Beaches and Dunes Classification
:-------------------------:|:-------------------------:
![](/images/cropReferenceMap.png)  |  ![](/images/cropBandD_classification.png)


 * Comparation

Changes           | Visualization of changes in the mosaic
:-------------------------:|:-------------------------:
![](/images/cropchanges.png)  |  ![](/images/cropmosaicChanges.png)
 * Legend:
      * Blue: Agreement
      * Green: Positive-Disagreement (B&D Classification  > Reference Map)


## Usage of the Source Script
* NECESSARY IMPORTS
```javascript
var table = ee.FeatureCollection("users/luizcf14/Artigo_Luize/ecossistemas_costeiros_maio2010"),
    imageVisParam = {"opacity":1,"bands":["swir1","nir","red"],"min":100,"max":143,"gamma":1},
    geometry = 
    ee.Geometry.Polygon(
        [[[-53.614819140625, -32.35235042278909],
          [-53.614819140625, -33.67429275769536],
          [-51.95588359375, -33.67429275769536],
          [-51.95588359375, -32.35235042278909]]], null, false);
}
```
* BASIC CONFIGURATIONS
```javascript
// Year selection
var year = 1985

// Basic mosaic visualization
// ATTENTION: PLEASE, DO NOT MODIFY THE FOLLOWING LINE
var mosaic = ee.Image("projects/samm/SAMM/Mosaic/"+year+"_v2")
Map.addLayer(mosaic,imageVisParam,'Mosaic')

// Blank canvas for better visualization of chances
Map.addLayer(ee.Image(0),{palette:'FFFFFF'},'Blank',false)
```

* VISUALIZATION OF MapBiomas 4.1 CLASSIFICATION
```javascript
// ATTENTION: PLEASE, DO NOT MODIFY THE FOLLOWING LINES
var merge = ee.Image('projects/mapbiomas-workspace/TRANSVERSAIS/ZONACOSTEIRA4_1-FT/'+year).eq(23).unmask(0)
var displacedMergeMask = merge.focal_max(4).reproject('EPSG:4326', null, 30)
merge = merge.updateMask(displacedMergeMask.eq(1)).unmask(0)
Map.addLayer(merge,{palette:['white','red'],min:0,max:1},'Reference Mapbiomas 4.1',false)
```

* SORT POINTS
```javascript
var points = merge.stratifiedSample(10000,'classification',geometry,30,null,1,[0,1],[5000,5000],true,1,true)
print(points.size())
print(points.first())
Map.addLayer(points,{},'Points',false)
```
* OUR BEACHES AND DUNES (B&D) CLASSIFICATION
```javascript
// ATTENTION: PLEASE, DO NOT MODIFY THE FOLLOWING LINES
var BD_Classification = ee.Image('projects/mapbiomas-workspace/TRANSVERSAIS/ZONACOSTEIRA5-FT/'+year+'-8')
```
* COMPARISON BETWEEN REFERENCE MAP (MapBiomas 4.1) AND B&D CLASSIFICATION
```javascript
points= points.map(function(feat){
  return feat.set({'BD_Classification':BD_Classification.eq(23).reduceRegion(ee.Reducer.first(),feat.geometry(),30).get('classification')})
})
print(points.first())

Map.addLayer(BD_Classification.mask(BD_Classification.eq(23)),{palette:'blue'},'B&D Classification',false)
}
```

* CORRELATIONS
```javascript
//No-change
var noChangeP = BD_Classification.eq(23).and(merge.eq(1))
var noChangeN = BD_Classification.neq(23).and(merge.eq(0))

//Positive-Disagreement (BD_Classification B&D -> ecosystem non-B&D)
var positiveDisagreement = BD_Classification.eq(23).and(merge.neq(1))

//Negative-Disagreement (BD_Classification non-B&D -> ecosystem B&D)
var nagativeDisagreement = BD_Classification.neq(23).and(merge.eq(1))

var changes = ee.ImageCollection([ee.Image(1).toByte().mask(noChangeP),
                                  ee.Image(2).toByte().mask(positiveDisagreement),
                                  ee.Image(3).toByte().mask(nagativeDisagreement)
                                  ]).max().unmask(0)
Map.addLayer(changes.mask(changes.gt(0)),{palette:['000000','0000FF','00FF00','FF0000'],min:0,max:3},'Changes')
}
```

* STATISTICS
```javascript
var statesBR = ee.FeatureCollection('users/luizcf14/Brasil/estados')
 var changesPerState = statesBR.map(function(feat){
      var nochangesP = noChangeP.reduceRegion({reducer:ee.Reducer.sum(),geometry:feat.geometry(),scale:30,maxPixels:1e13})
      var nochangesN = noChangeN.reduceRegion({reducer:ee.Reducer.sum(),geometry:feat.geometry(),scale:30,maxPixels:1e13})
      var positive = positiveDisagreement.reduceRegion({reducer:ee.Reducer.sum(),geometry:feat.geometry(),scale:30,maxPixels:1e13})
      var negative = nagativeDisagreement.reduceRegion({reducer:ee.Reducer.sum(),geometry:feat.geometry(),scale:30,maxPixels:1e13})
      var sigla = feat.get('sigla')
    return ee.Feature(null,{'nochangeN':nochangesN.get('classification'),'nochangeP':nochangesP.get('classification'),'positive':positive.get('classification'),'negative':negative.get('classification'),'sigla':sigla})
 })
var changesPerState_Exemple = changesPerState.first()
print('Brasilian state exemple: ', changesPerState_Exemple.get('sigla'), changesPerState_Exemple) 
Export.table.toDrive(changesPerState,'changes_per_state_'+year+'_mapbiomas','results_BandD','changes_per_state_'+year+'_mapbiomas')

```

* LOCALIZATION SET
```javascript
Map.setCenter(-52.5052,-32.8429,12)
```

* POINTS BASED STATS
```javascript
var errorM = points.errorMatrix('classification','BD_Classification')
print('Error Matrix',errorM)
print('O.A',errorM.accuracy())
print('Kappa',errorM.kappa())
print('Consumer',errorM.consumersAccuracy())
print('Producers',errorM.producersAccuracy())

```
## Supplementary Material
For more information about de results obtained from the classification, graphics and statistical analysis, please consult the excel file [complete_Results_Brazil(1985-2019).xlsx](https://github.com/MariaLuize/Brazilian-Beaches-and-Dunes/blob/5243b1dd5514dfea560032cb8acec1b352f5a94b/Supplementary%20Material/complete_Results_Brazil(1985-2019).xlsx). It is possible to consult the complete data obtained from the methodology presented in the presented article.
