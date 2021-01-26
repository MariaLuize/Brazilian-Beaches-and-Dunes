# Brazilian-Beaches-and-Dunes

Visualization of final classification and statistics: https://code.earthengine.google.com/11a49b8d9773c13faf909f55f813e80c?noload=1

Visual Results
* Base Mosaic
![](/images/cropbaseMosaic.png)

 * Classification Maps
 
Reference Map - MapBiomas 4.1 Classification             | Beaches and Dunes Classification]
:-------------------------:|:-------------------------:
![](/images/cropReferenceMap.png)  |  ![](/images/cropBandD_classification.png)


 * Comparation

Changes           | Visualization of changes in the mosaic
:-------------------------:|:-------------------------:
![](/images/cropchanges.png)  |  ![](/images/cropmosaicChanges.png)
 * Legend:
      * Blue: Agreement
      * Green: Positive-Disagreement (B&D Classification  > Reference Map)

```javascript

//IMPORTS
var table = ee.FeatureCollection("users/luizcf14/Artigo_Luize/ecossistemas_costeiros_maio2010"),
    imageVisParam = {"opacity":1,"bands":["swir1","nir","red"],"min":100,"max":143,"gamma":1},
    geometry = 
    /* color: #d63000 */
    /* shown: false */
    /* displayProperties: [
      {
        "type": "rectangle"
      }
    ] */
    ee.Geometry.Polygon(
        [[[-53.614819140625, -32.35235042278909],
          [-53.614819140625, -33.67429275769536],
          [-51.95588359375, -33.67429275769536],
          [-51.95588359375, -32.35235042278909]]], null, false);
          
}
```
```javascript
// YEAR SELECTION
var year = 1985
var mosaic = ee.Image("projects/samm/SAMM/Mosaic/"+year+"_v2")

// BASIC MOSAIC VISUALIZATION
Map.addLayer(mosaic,imageVisParam,'Mosaic')

// BLANK CANVAS FOR BETTER VISUALIZATION
Map.addLayer(ee.Image(0),{palette:'FFFFFF'},'Blank',false)

// REFERENCE MAP - MapBiomas 4.1
var merge = ee.Image('projects/mapbiomas-workspace/TRANSVERSAIS/ZONACOSTEIRA4-FT/'+year).eq(23).unmask(0)
var displacedMergeMask = merge.focal_max(4).reproject('EPSG:4326', null, 30)
merge = merge.updateMask(displacedMergeMask.eq(1))
Map.addLayer(merge,{palette:['white','red'],min:0,max:1},'Reference Mapbiomas 4.1',false)
```
