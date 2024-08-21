---
title: Useful Abaqus Commands
categories: [FEA, Abaqus]
tags: [Abaqus, Python]
render_with_liquid: false
---


Abaqus is a commonly used finite element suite and it comes packaged with python **2.7**.  Here are several useful commands for model creation and processing.

# Coordinates in place of Index
Using default journal options, Abaqus defines its objects by index, called native masks. The following snippet instead tells Abaqus to store that information as coordinates. 
```python
session.journalOptions.setValues(replayGeometry=COORDINATE, recoverGeometry=COORDINATE)
```

# Libraries
To create a Abaqus model from scratch, the following libraries are needed. 
```python
from abaqus import *
from abaqusConstants import *
from caeModules import *
from driverUtils import executeOnCaeStartup
executeOnCaeStartup()
session.viewports['Viewport: 1'].partDisplay.geometryOptions.setValues(
    referenceRepresentation=ON)
import numpy as np
```

# Script Executions
Abaqus can be called from the terminal with ```abaqus cae```. For complete models, ```noGUI``` runs Abaqus in the background. 
```console
abaqus cae script=EXAMPLE.py 
```
```console
abaqus cae noGUI=EXAMPLE.py
```
# Images
The following snippets generate images for publications.

## Simple screenshots
For simple screenshots, the following snippet disables all viewport objects, and sets the background to white.
```python
session. linkedViewportCommands.setValues(linkViewports=True)
session.graphicsOptions.setValues(backgroundStyle=SOLID, backgroundColor='#FFFFFF')
session.viewports['Viewport: 1'].viewportAnnotationOptions.setValues(triad=OFF,	legend=OFF, title=OFF, state=OFF, annotations=OFF, compass=OFF)
```

## Publication Images
For higher quality images, the following exports a render from Abaqus directly:
```python
myFile = 'ExamplePicture' # extension not needed, saves to the working directory
session.printToFile(fileName=myFile, format=PNG, canvasObjects=(
    session.viewports['Viewport: 1'], ))
```

## Deformed Geometry
You can export the deformed geometry using the following script:
```python
fileName = './name.obj' # extension needed, saves to the working directory
session.writeOBJFile(fileName=fileName, 
	canvasObjects=(session.viewports['Viewport: 1'], )) # Exports the main window
```
This script can be extended to export a sequence of geometries for a each frame in a ODB. However, for complex models this can result in several terabytes of models being generated, so it is not recommended. Instead, I would suggest linking a modeling program like Blender to read from and ODB to dynamically create a render of the model. 

## Set font to Computer Modern (LaTeX font)

Assuming you have computer modern installed:
```python
fontName = 'cmu serif-medium-r-normal'
fontSize = 240 # for 24pt, 0 is the decimal precision. Default is 12pt.
session.viewports['Viewport: 1'].viewportAnnotationOptions.setValues(
    legendFont='-*-'+fontName+'-*-*-'+str(fontSize)+'-*-*-p-*-*-*')
```
Replace ```fontName``` with your target font, common ones:

| Font Name | Command |
|--|--|
|Times New Roman | ```times new roman-medium-r-normal``` |
|Arial | ```arial-medium-r-normal``` |
|Objectively the best | ```comic sans ms-medium-r-normal``` |

The information after the font name refers to other characteristics such as italicized. I don't recommend modifying these extensions manually.