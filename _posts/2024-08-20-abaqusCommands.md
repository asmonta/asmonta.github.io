---
title: Abaqus Scripting Quickstart
categories: [FEA, Abaqus]
tags: [Abaqus, Python]
render_with_liquid: false
---


# Useful Abaqus Scripting Snippets

Abaqus is a commonly used finite element suite and it comes packaged with Python **2.7**. Here are several useful commands for model creation, automation, and postprocessing.

---

## Basic Snippets

### Coordinates in Place of Index

Using default journal options, Abaqus defines its objects by index, called native masks. The following snippet instead tells Abaqus to store that information as coordinates.

```python
session.journalOptions.setValues(replayGeometry=COORDINATE, recoverGeometry=COORDINATE)
```

---

### Script Executions

Abaqus can be called from the terminal with `abaqus cae`. For complete models, `noGUI` runs Abaqus in the background.

```console
abaqus cae script=EXAMPLE.py 
```

```console
abaqus cae noGUI=EXAMPLE.py
```

---

### Images

The following snippets generate images for publications.

#### Simple Screenshots

Disables all viewport objects and sets the background to white.

```python
session.linkedViewportCommands.setValues(linkViewports=True)
session.graphicsOptions.setValues(backgroundStyle=SOLID, backgroundColor='#FFFFFF')
session.viewports['Viewport: 1'].viewportAnnotationOptions.setValues(
    triad=OFF, legend=OFF, title=OFF, state=OFF, annotations=OFF, compass=OFF)
```

#### Publication Images

For high-quality renders from Abaqus:

```python
myFile = 'ExamplePicture'  # extension not needed, saves to the working directory
session.printToFile(fileName=myFile, format=PNG, canvasObjects=(
    session.viewports['Viewport: 1'], ))
```

#### Deformed Geometry

Export deformed shape to `.obj` format:

```python
fileName = './name.obj'
session.writeOBJFile(fileName=fileName, 
    canvasObjects=(session.viewports['Viewport: 1'], ))  # Exports the main window
```

This can be extended to loop through all frames in an ODB, but the file sizes grow quickly. For complex workflows, consider using a dedicated rendering tool (e.g. Blender) linked to ODB output.

---

#### Font Customization for Figures

Set font to Computer Modern (LaTeX style):

```python
fontName = 'cmu serif-medium-r-normal'
fontSize = 240  # for 24pt
session.viewports['Viewport: 1'].viewportAnnotationOptions.setValues(
    legendFont='-*-'+fontName+'-*-*-'+str(fontSize)+'-*-*-p-*-*-*')
```

Replace `fontName` with one of the following:

| Font Name            | Command                           |
| -------------------- | --------------------------------- |
| Times New Roman      | `times new roman-medium-r-normal` |
| Arial                | `arial-medium-r-normal`           |
| Objectively the best | `comic sans ms-medium-r-normal`   |

---

### Named Sets and Surfaces from Geometry

Create named regions for reuse later in BCs or loads:

```python
face = part.faces.findAt(((0.0, 0.0, 0.0),))
part.Set(name='FaceSet', faces=face)

edge = part.edges.findAt(((0.0, 1.0, 0.0),))
part.Surface(name='TopEdge', side1Edges=edge)
```

---

### Material Assignment to All Faces

```python
faces = part.faces
region = regionToolset.Region(faces=faces)
part.SectionAssignment(region=region, sectionName='Section-1')
```

---

### Monitor Job Status Programmatically

Wait for a job to finish before proceeding:

```python
mdb.jobs['Job-1'].submit()
mdb.jobs['Job-1'].waitForCompletion()
```

Check job status:

```python
if mdb.jobs['Job-1'].status == COMPLETED:
    print("Job completed.")
```

---

### Mesh Controls with Specific Element Types

Example for assigning C3D8R elements:

```python
elemType1 = mesh.ElemType(elemCode=C3D8R, elemLibrary=STANDARD)
region = (part.cells,)
part.setElementType(regions=region, elemTypes=(elemType1,))
```

---

### Sweep Mesh with Medial Axis

```python
part.setMeshControls(regions=part.cells, technique=SWEEP, algorithm=MEDIAL_AXIS)
```

---

### Batch Postprocessing Across Jobs

Loop through multiple `.odb` files:

```python
from odbAccess import openOdb

for job in ['Job1', 'Job2', 'Job3']:
    odb = openOdb(f'{job}.odb')
    step = odb.steps['Step-1']
    frame = step.frames[-1]
    disp = frame.fieldOutputs['U']
    max_disp = max(v.magnitude for v in disp.values)
    print(f'{job}: Max displacement = {max_disp:.4e}')
    odb.close()
```

---

### Suppress Console Warnings

Cleaner log output for batch processing:

```python
import warnings
warnings.filterwarnings("ignore")
```

---

### Field Math: Compare Frames

Compare field results between two time steps:

```python
frame1 = odb.steps['Step-1'].frames[0]
frame2 = odb.steps['Step-1'].frames[-1]

u1 = frame1.fieldOutputs['U']
u2 = frame2.fieldOutputs['U']
du = u2 - u1  # Returns a new FieldOutput object
```

---

### Disable GUI Features to Speed Up CAE

```python
session.journalOptions.setValues(defaultValues=OFF)
session.graphicsOptions.setValues(cadDatumVisibility=False)
```

---

### Main Script Entry Point

This avoids running your whole script when importing it into another file:

```python
def main():
    # build model or postprocess
    pass

if __name__ == '__main__':
    main()
```

---

## Advanced Snippets

For users working on complex workflows, large batches, or GUI automation, the following advanced snippets may be useful.

---

### Multiprocessing for Batch Runs

Use Python’s `multiprocessing` module to parallelize batch simulations. This works well if you’re launching separate Abaqus processes for each job.

```python
import multiprocessing as mp
import os

def run_job(width):
    os.system(f'abaqus cae noGUI=your_script.py -- {width}')

if __name__ == '__main__':
    widths = [5, 10, 15, 20]
    with mp.Pool(processes=4) as pool:
        pool.map(run_job, widths)
```

In `your_script.py`, parse the width from `sys.argv`:

```python
import sys
width = float(sys.argv[-1])  # last item is the passed argument
```

---

### Adding a GUI Form with a Custom Plug-in

Abaqus supports custom plug-ins using the `toolset` and `AFX` GUI libraries. Here’s a minimal GUI example:

```python
from abaqusGui import *
from kernelAccess import mdb

class MyDialog(AFXDataDialog):
    def __init__(self, form):
        AFXDataDialog.__init__(self, form, 'My Plug-in Dialog')
        self.widthKw = AFXFloatKeyword(form, 'width', True)
        AFXTextField(p=self, ncols=10, labelText='Width:', tgt=self.widthKw, sel=0)

class MyPlugin(AFXForm):
    def __init__(self, owner):
        AFXForm.__init__(self, owner)
    def getFirstDialog(self):
        return MyDialog(self)
    def doCustomChecks(self):
        return True
    def okPressed(self):
        width = self.widthKw.getValue()
        print(f'Width entered: {width}')

thisForm = MyPlugin(getAFXApp())
```

To load it:
**Plug-ins → Abaqus Plug-in Manager → Register Plug-in**

---

### Scriptable Model Comparison

Similar to odb comparisons, you can compare geometry between two models:

```python
p1 = mdb.models['Model-1'].parts['Part-1']
p2 = mdb.models['Model-2'].parts['Part-1']

if len(p1.nodes) != len(p2.nodes):
    print("Models differ in node count.")
```

That said, I typically don't recommend having more than one model active though to prevent simple oversights that can lead to difficult to diagnose errors, but multiple model databases within a model has its place.  
---

### Link with External Python Libraries

Though Abaqus uses Python 2.7, many libraries (e.g., NumPy, matplotlib) are still usable if installed in the Abaqus Python environment.

```python
import numpy as np
data = np.loadtxt('results_summary.csv', delimiter=',', skiprows=1)
print("Average displacement:", np.mean(data[:,1]))
```

If needed, point Abaqus to your own Python install, or use subprocesses to hand data off to an external Python 3+ workflow.

---

### Automatically Save/Close Files in GUI Mode

When scripting inside CAE with GUI:

```python
session.saveAs(pathName='my_model.cae')
session.odbs['Job-1.odb'].close()
```

---

### Suppress Graphical Output for Scripting Only

If you want to suppress pop-ups and GUI updates during heavy scripting:

```python
session.journalOptions.setValues(replayGeometry=COORDINATE)
session.viewports['Viewport: 1'].makeCurrent()
session.viewports['Viewport: 1'].disableGraphics()
```