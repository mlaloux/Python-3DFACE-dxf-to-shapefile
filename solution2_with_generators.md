Pour éviter de devoir charger la couche en mémoire avec la première solution:   


Générateur/itérateur

`
```python

       def readfile(filepath, delim1, delim2): 
            a = []
            copy = False
            with open(filepath, 'r') as infile: 
               for line in infile:
                   if line.strip() == delim1:
                        copy = True
                        if a:yield a
                        a = []
                   elif line.startswith(delim2):
                        copy = False
                   elif copy:
                        a.append(line.strip())
 
```

Traitement (ne marche pas avec Pyshp):

```python

       import fiona
       from shapely.geometry import Polygon, mapping
       schema = {'geometry': '3D Polygon','properties': {'name': 'str'}}
       with fiona.open('tokajpoly.shp','w','ESRI Shapefile', schema) as e:  
           for line in rea``pythondfile("tokaj.dxf","3DFACE","  0"):
                name,x1,y1,z1,x2,y2,z2,x3,y3,z3,x4,y4,z4 = line[1::2]
                geom = Polygon([(float(x1),float(y1),float(z1)), (float(x2),float(y2),float(z2)), (float(x3),float(y3),float(z3))]) 
                e.write({'geometry':mapping(geom), 'properties':{'name':name}})
                
```

<img src="http://i.imgur.com/8DN7hlV.jpg" HR WIDTH="60%">

Comme il y a des lignes dans le fichier dxf (LINES)

```python
      schema = {'geometry': '3D LineString','properties': {'name': 'str'}}
       with fiona.open('tokajline.shp','w','ESRI Shapefile', schema) as e:  
           for line in rea``pythondfile("tokaj.dxf","3DFACE","  0"):
                name,x1,y1,z1,x2,y2,z2 = line[1::2]
                geom = LineString([(float(x1),float(y1),float(z1)), (float(x2),float(y2),float(z2))])  
                e.write({'geometry':mapping(geom), 'properties':{'name':name}})
                
```
<img src="http://i.imgur.com/uz7k0mH.jpg" HR WIDTH="60%">
