avec le format dxf de  [ 3DFACE](http://www.autodesk.com/techpubs/autocad/acad2000/dxf/3dface_dxf_06.htm) et en passant par un dictionnaire (pour éviter les 0 comme délimiteurs)

###3D faces 

```python
def readface(filepath, delim): 
      face = []
      go = False
      print delim
      with open(filepath, 'r') as infile: 
         for line in infile:
             if line.strip() == delim:
                  go = True
                  if face:yield dict(zip(face[0::2], face[1::2]))
                  face = []
             elif go:
                   if len(face) < 27:face.append(line.strip())
         yield dict(zip(face[0::2], face[1::2]))
```

Traitement

 - Avec Pyshp  
 
```python
import shapefile
w = shapefile.Writer(shapeType=shapefile.POLYGONZ)
w.field("NAME")
for line in readface("tokaj.dxf","3DFACE"):
      w.poly([[[float(line['10']),float(line['20']),float(line['30'])], [float(line['11']),float(line['21']),float(line['31'])], [float(line['13']),float(line['23']),float(line['33'])]]])
      w.record(line['8'])

w.save("tokaj4.shp")
```
 
 - Avec Fiona et shapely  

```python
import fiona
from shapely.geometry import Polygon, mapping
schema = {'geometry': '3D Polygon','properties': {'name': 'str'}}
with fiona.open('tokaj6.shp','w','ESRI Shapefile', schema) as e:  
    for line in readfile("tokaj.dxf","3DFACE"):
        geom = Polygon([(float(line['10']),float(line['20']),float(line['30'])), (float(line['11']),float(line['21']),float(line['31'])), (float(line['12']),float(line['22']),float(line['32'])),(float(line['13']),float(line['23']),float(line['33']))])    
        attr = line['8']
        e.write({'geometry':mapping(geom), 'properties':{'name':attr}})
        
```

###lines

```python
def readline(filepath, delim): 
      linecomp = []
      go = False
      print delim
      with open(filepath, 'r') as infile: 
         for line in infile:
             if line.strip() == delim:
                  go = True
                  if linecomp:yield dict(zip(linecomp[0::2], linecomp[1::2]))
                  linecomp = []
             elif copy:
                  if len(linecomp) < 15: linecomp.append(line.strip())
         yield dict(zip(linecomp[0::2], linecomp[1::2]))

```


Traitement


```python
schema = {'geometry': '3D LineString','properties': {'name': 'str'}}
with fiona.open('tokajline3.shp','w','ESRI Shapefile', schema) as e:  
    for line in readline("tokaj.dxf","LINE"):
         geom = LineString([(float(line['10']),float(line['20']),float(line['30'])), (float(line['11']),float(line['21']),float(line['31']))])  
         attr = line['8']          
         e.write({'geometry':mapping(geom), 'properties':{'name':name}})
         
```
