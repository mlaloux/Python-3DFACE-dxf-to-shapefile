
In the dxf file, A 3D face polygon is located between the two delimiters:

3DFace  
.....  
  0  
(the spaces before the 0 are important here, because there are also coordinates equal to 0, without spaces)

I  use the solution of Brent Newey in [Repeatedly extract a line between two delimiters in a text file, Python](http://stackoverflow.com/questions/7098530/repeatedly-extract-a-line-between-two-delimiters-in-a-text-file-python)

```python
                re.findall('DELIMITER1(.*?)DELIMITER2'
```

I had to try many many regular expressions to  extract the polygons but this one works with all my examples:

```python
                 re.findall(''3DFACE(.*?)\s{3}0'')
```

The main drawback of this approach is that the dxf file must be read in memory entirely before it's used (but no problem with the 33.3 Mo of Otway_Basin_Granites-gravity.dxf)

The scripts:

#####1) if you want a 3D shapefile as a result, you can use pyshp, Fiona and shapely or osgeo.ogr (PyQGIS  does not support 3D)

With pyshp:

```python

       import shapefile
       import re
       w = shapefile.Writer(shapeType=shapefile.POLYGONZ)
       with open("tokaj.dxf") as fp:
       w.field("NAME")
       for result in re.findall('3DFACE(.*?)\s{3}0', fp.read(), re.S):
              a = result.split()
              name,x1,y1,z1,x2,y2,z2,x3,y3,z3,x4,y4,z4 = a[1::2]
              w.poly([[[float(x1),float(y1),float(z1)], [float(x2),float(y2),float(z2)], [float(x3),float(y3),float(z3)]]])
              w.record(name)

       w.save("tokaj.shp")
```

With Fiona and shapely:

```python

       import fiona
       from shapely.geometry import Polygon, mapping
       import re
       schema = {'geometry': '3D Polygon','properties': {'name': 'str'}}
       with open("tokaj.dxf") as fp:
            with fiona.open('tokaj2.shp','w','ESRI Shapefile', schema) as e:  
                for result in re.findall('3DFACE(.*?)\s{3}0', fp.read(), re.S):
                   a = result.split()
                   name,x1,y1,z1,x2,y2,z2,x3,y3,z3,x4,y4,z4 = a[1::2]
                   geom = Polygon([(float(x1),float(y1),float(z1)), (float(x2),float(y2),float(z2)), (float(x3),float(y3),float(z3))])            
                    e.write({'geometry':mapping(geom), 'properties':{'name':name}})
                    
```

#####2) if you want directly a 2D shapefile in QGIS, you can use the Python console:

```python

       from PyQt4.QtCore import *
       import re
       fields = QgsField("name", QVariant.String)
       layer = QgsVectorLayer("Polygon", "tokaj", "memory")
       pr = layer.dataProvider()
       pr.addAttributes([QgsField("name", QVariant.String)])
       with open("tokaj.dxf") as fp:
            for result in re.findall('3DFACE(.*?)\s{3}0', fp.read(), re.S):
                 a = result.split()
                 name,x1,y1,z1,x2,y2,z2,x3,y3,z3,x4,y4,z4 = a[1::2]
                 fet = QgsFeature()
                 fet.setAttributes([name])
                 geom = QgsGeometry.fromPolygon([[QgsPoint(float(x1),float(y1)), QgsPoint(float(x2),float(y2)), QgsPoint(float(x3),float(y3))]])
                 fet.setGeometry(geom)
                 pr.addFeatures([fet])
                 layer.updateExtents()
             
         QgsMapLayerRegistry.instance().addMapLayers([layer]) 
         
```

Result:

![Result][1]


  [1]: http://osgeo-org.1560.x6.nabble.com/file/n5095133/tokaj.jpg
