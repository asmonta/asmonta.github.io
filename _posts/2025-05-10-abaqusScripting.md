---
title: Abaqus Scripting Overview
categories: [FEA, Abaqus]
tags: [Abaqus, Python]
render_with_liquid: false
---

# Abaqus Scripting Overview

## Introduction

Abaqus ships with python **2.7** and allows you to automate, pre- and post-process, streamline parametric studies, and batch-run complex simulations. This page provides a brief overview of its scripting abilities while providing some common snippets. 

**Official Docs**: [SIMULIA Documentation Portal](https://www.3ds.com/support/documentation)

[**Useful Commands**](https://www.3ds.com/support/documentation)

---

## 1. Basic Concepts

### 1.1 What is Abaqus scripting?

Abaqus scripting is the use of Python (with custom Abaqus modules) to programmatically access and manipulate the model database (`.cae`) and output database (`.odb`). This is done entirely within Abaqus, and enables:

* Model creation and modification
* Job submission and monitoring
* Postprocessing and report generation
* Parametric and batch studies

### 1.2 Running scripts

There are two standard methods for running a script:

* **Through command prompt**
  `abaqus cae script=myscript.py`

* **Within Abaqus**
  `File -> Run Script`
  or "Run Script" on welcome window. 

---

### 1.3 Imports and modules

A typical Abaqus script imports the following:

```python
from part import *
from material import *
from section import *
from assembly import *
from step import *
from interaction import *
from load import *
from mesh import *
from optimization import *
from job import *
from sketch import *
from visualization import *
from connectorBehavior import *
```

### 1.4 Broad script types:
An Abaqus model can be divided into two parts, a model database (MBD) that contains the geometry and like, and an output database (ODB) which stores results. While obviously you can make a script that both builds and processes a model, as a *generality* it is better to split your scripts into ones that automate model creation and ones that postprocess for readability, refactoring, and reusability. 

---

## 2. Model Building Workflow

A simplified typical modeling flow in scripts mirrors the GUI structure:

1. Create part
2. Define materials and sections
3. Assemble parts
4. Create steps
5. Define boundary conditions and loads
6. Mesh the part
7. Submit the job

### Example: Basic 2D model setup

```python
mdb.Model(name='Model-1')
model = mdb.models['Model-1']

sketch = model.ConstrainedSketch(name='profile', sheetSize=20.0)
sketch.rectangle(point1=(-5.0, -1.0), point2=(5.0, 1.0))

mdb.models['Model-1'].Part(dimensionality=TWO_D_PLANAR, 
                           name='Part-1', 
                           type=DEFORMABLE_BODY)
part.BaseShell(sketch=sketch)
```

### Material and section definition

```python
model.Material(name='Material-1')
model.materials['Material-1'].Elastic(table=((1e9, 0.3),))

model.HomogeneousSolidSection(name='Section-1', material='Material-1')

faces = part.faces.findAt(((0.0, 0.0, 0.0),))
region = regionToolset.Region(faces=faces)
part.SectionAssignment(region=region, sectionName='Section-1')
```

### Assembly and step creation

```python
assembly = model.rootAssembly
assembly.DatumCsysByDefault(CARTESIAN)
assembly.Instance(name='Part-1-1', part=part, dependent=ON)

model.StaticStep(name='Step-1', previous='Initial',
                 nlgeom=ON, initialInc=0.1, maxInc=0.1)
```

### Boundary conditions and loads

```python
edge = part.edges.findAt(((-5.0, 0.0, 0.0),))
part.Set(name='Set-1', edges=edge)
instance = assembly.instances['Part-1-1']
region = instance.sets['Set-1']

model.DisplacementBC(name='BC-1', createStepName='Step-1', 
    region=region, u1=0.0, u2=0.0)

top_edge = part.edges.findAt(((0.0, 1.0, 0.0),))
part.Surface(name='Surf-1', side1Edges=top_edge)
surface = instance.surfaces['Surf-1']

model.Pressure(name='Load-1', createStepName='Step-1',
    region=surface, magnitude=-1e-5)
```

### Mesh and job creation

```python
part.seedPart(size=0.5)
part.setMeshControls(regions=faces, technique=STRUCTURED, elemShape=QUAD)
elemType1 = mesh.ElemType(elemCode=CPS8R, elemLibrary=STANDARD)
elemType2 = mesh.ElemType(elemCode=CPS6M, elemLibrary=STANDARD)
part.setElementType(regions=faces, elemTypes=(elemType1, elemType2))
part.generateMesh()

mdb.Job(name='Job-1', model='Model-1')
mdb.jobs['Job-1'].submit()
```

All together, a rough template for an entire Abaqus model in script form is:
## Template Script Structure

```python
from abaqus import *
from abaqusConstants import *
from part import *  # ...etc

def build_model():
    # create model, part, material, section
    pass

def apply_loads_and_bcs():
    # create steps, apply BCs and loads
    pass

def mesh_and_submit():
    # mesh and submit job
    pass

if __name__ == '__main__':
    build_model()
    apply_loads_and_bcs()
    mesh_and_submit()
```

---

## 3. Parametric Studies

Rather than manually adjusting the parameters or hyperparameters of a model, these values can be dynamically or programmatically controlled using scripts. Abaqus scripting is particularly well suited to this task, allowing you to loop over geometry, material properties, boundary conditions, loads, or even entire models to systematically generate variations and explore a design space.

This approach is especially useful for parametric sweeps, "what-if" analyses, and optimization routines.

Using the previous modeling structure as a base, a parametric study can be implemented by embedding key variables (e.g., width) into the model definition. The script then loops over a defined range of values, generating and running each variation automatically:

```python
def build_model(width):
    # create model, part, material, section
    pass

for width in range(widthMin, widthMax):
    build_model(width)
```

As an example, this script automates a study of beam deflection with different widths.

```python
from abaqus import *
from abaqusConstants import *
from part import *
from material import *
from section import *
from assembly import *
from step import *
from load import *
from mesh import *
from job import *

def create_model(width):
    model_name = f'Model_{width}'
    job_name = f'Job_{width}'

    model = mdb.Model(name=model_name)
    s = model.ConstrainedSketch(name='beam_profile', sheetSize=200.0)
    s.rectangle(point1=(0, 0), point2=(width, 10.0))

    part = model.Part(name='Beam', dimensionality=TWO_D_PLANAR,
                      type=DEFORMABLE_BODY)
    part.BaseShell(sketch=s)

    model.Material(name='Steel')
    model.materials['Steel'].Elastic(table=((210e9, 0.3),))
    model.HomogeneousSolidSection(name='Section', material='Steel')

    region = part.faces.findAt(((width / 2.0, 5.0, 0.0),))
    part.SectionAssignment(region=region, sectionName='Section')

    assembly = model.rootAssembly
    assembly.DatumCsysByDefault(CARTESIAN)
    inst = assembly.Instance(name='BeamInst', part=part, dependent=ON)

    model.StaticStep(name='LoadStep', previous='Initial')

    edge = part.edges.findAt(((0.0, 5.0, 0.0),))
    part.Set(name='FixedEdge', edges=edge)
    assembly.Set(name='FixedEdgeInst', 
                 referencePoints=(inst.edges.findAt(((0.0, 5.0, 0.0),))[0],))

    model.DisplacementBC(name='FixedBC', createStepName='LoadStep',
        region=assembly.instances['BeamInst'].sets['FixedEdge'],
        u1=0.0, u2=0.0)

    edge = part.edges.findAt(((width, 5.0, 0.0),))
    part.Surface(name='LoadSurf', side1Edges=edge)
    model.Pressure(name='EndLoad', createStepName='LoadStep',
        region=assembly.instances['BeamInst'].surfaces['LoadSurf'],
        magnitude=-1e6)

    part.seedPart(size=1.0)
    part.setMeshControls(regions=part.faces, technique=STRUCTURED)
    elemType = mesh.ElemType(elemCode=CPS4R, elemLibrary=STANDARD)
    part.setElementType(regions=part.faces, elemTypes=(elemType,))
    part.generateMesh()

    mdb.Job(name=job_name, model=model_name)
    mdb.jobs[job_name].submit()
    mdb.jobs[job_name].waitForCompletion()

# Parametric loop
for w in [5.0, 10.0, 15.0, 20.0]:
    create_model(width=w)
```

---

### Notes and best practices

* **Use functions**: Encapsulate model creation logic in functions for clarity and reuse.
* **Wait for job completion** before postprocessing with `.waitForCompletion()`.
* Make unique jobnames to prevent simple oversights which can cause errors.
* Execute scripts without gui (`abaqus cae noGUI=script.py`) to increase performance.
* Clean up models if needed using `del mdb.models[model_name]` to save memory.


## 4. Postprocessing

Once a model has been executed, its simulation results are stored in an accompanying `.odb` file and can be parsed to obtain specific information or to translate data into other formats.

### 4.1 ODB Structure

An `.odb` file has a hierarchical structure:

```
odb
 └── steps
      └── Step-1
           ├── frames (output frames over time)
           ├── fieldOutputs (U, S, etc.)
           └── historyRegions
      └── Step-2 ...
```

* **Steps** represent analysis increments (e.g. load steps)
* **Frames** contain data snapshots at particular increments/iterations
* **Field outputs** include spatial fields (e.g. stress, displacement)
* **History outputs** include scalar values tracked over time (e.g. reaction forces)


---

### 4.2 Opening the Output Database

Use the `openOdb` function from `odbAccess` or `session.openOdb()` to load the `.odb`.

```python
from odbAccess import openOdb
odb = openOdb(path='Job-1.odb')
```

To display it in CAE (optional):

```python
session.viewports['Viewport: 1'].setValues(displayedObject=odb)
```

To close it:

```python
odb.close()
```

---

### 4.3 Accessing Field Output

The `.odb` object stores simulation results and can be queried in a familiar Pythonic manner. For example, the list of steps in the model can be obtained from:

```python
print(odb.steps.keys())
```

Field outputs are spatial—defined at nodes or integration points with relevant information obtained from the model. The most common ones are stress (`'S'`), displacment (`'U'`), reaction force (`'RF'`), strain (`'E'`), and logarithmic strain (`'LE'`). These can be found by:

```python
frame = odb.steps['Step-1'].frames[-1]
print(frame.fieldOutputs.keys())  # e.g., 'S'-stress, 
                                  #       'U'-displacment, 
                                  #       'RF'-reaction force, 
                                  #       'E'-strain, 
                                  #       'LE'-log. strain
```

Using displacement as an example, the displacement of the nodes of the model can be found with:

```python
disp = frame.fieldOutputs['U']  # Displacement
for v in disp.values:
    print(f'Node {v.nodeLabel}: U1={v.data[0]}, U2={v.data[1]}')
```

Information pertaining to a specific node can be found with:
```python
node_disp = [v for v in disp.values if v.nodeLabel == 100][0]
print(node_disp.data)
```

This data can be used or maniuplated in the typical manor. For example, you can subtract or combine fields using standard operators to perform custom calculations like incremental displacement, stress changes, etc.:

```python
frame1 = odb.steps['Step-1'].frames[-2]
frame2 = odb.steps['Step-1'].frames[-1]
disp1 = frame1.fieldOutputs['U']
disp2 = frame2.fieldOutputs['U']
delta_disp = disp2 - disp1
```

One common use is to print the data into a neutral format, such as `.csv`:

```python
with open('displacement_output.csv', 'w') as f:
    f.write('Node,U1,U2,U3\n')
    for v in disp.values:
        u = v.data
        f.write(f'{v.nodeLabel},{u[0]},{u[1]},{u[2]}\n')
```

Integration points are parsed in a similar manor:

```python
stress = frame.fieldOutputs['S']
for v in stress.values:
    if v.integrationPoint:
        print(f'Elem {v.elementLabel}: von Mises = {v.mises}')
```
---

### 4.4 Accessing History Output

History outputs, unlike field data, represent scalar values tracked over time—e.g., force or displacement at specific nodes or regions. Regardless, they are parsed in a similar manor to field objects

#### Accessing reaction force:

```python
region = odb.steps['Step-1'].historyRegions
for key in region.keys():
    print(key)  # e.g., 'Node PART-1-1.100'

rf = region['Node PART-1-1.100'].historyOutputs['RF2']
for time, val in rf.data:
    print(f'Time={time}, RF2={val}')
```

#### Exporting to CSV:

```python
with open('rf_vs_time.csv', 'w') as f:
    f.write('Time,RF2\n')
    for time, val in rf.data:
        f.write(f'{time},{val}\n')
```
---

### 4.5 Working with Sets and Regions

As an alternative to node definitions, sets can be used to make automation more friendly, but follow a similar pattern:

```python
part_instance = odb.rootAssembly.instances['PART-1-1']
region = part_instance.nodeSets['SET-1']
```

Then:

```python
disp = frame.fieldOutputs['U'].getSubset(region=region)
```

You can create sets in your model and access them by name in postprocessing scripts.

---

### 4.6 Plotting in CAE

Results can be visualized either as colored contour plots on the model geometry or as XY plots. Personally, I find extracting results to `.csv` and using external tools more effective, but both approaches are scriptable in Abaqus.

#### Contour plot of displacement:

```python
session.viewports['Viewport: 1'].odbDisplay.setPrimaryVariable(
    field=disp, outputPosition=NODAL)
session.viewports['Viewport: 1'].odbDisplay.display.setValues(
    plotState=(CONTOURS_ON_DEF,))
```

#### XY plot of displacement over time:

```python
xy_data = xyPlot.XYDataFromHistory(name='Disp_vs_Time',
    odb=odb, outputVariableName='U2 at Node 100',
    steps=('Step-1',),
    region=region['Node PART-1-1.100'],
    variable='U2')

session.xyPlots['MyPlot'] = xyPlot.XYPlot('MyPlot')
c = session.xyPlots['MyPlot'].chartNames[0]
chart = session.xyPlots['MyPlot'].charts[c]
chart.setValues(curves=(session.Curve(xyData=xy_data),))

session.viewports['Viewport: 1'].setValues(displayedObject=session.xyPlots['MyPlot'])
```

### 4.7 Multiple ODBs

When run in a batch, rather than processing the models in-loop, they can all be run, then accessed after. This process is straightforward and similar to the previous examples:

```python
from odbAccess import openOdb
from abaqusConstants import *

for job_name in ["Job_1", "Job_2", "Job_3"]:
    odb = openOdb(f'{job_name}.odb')
    last_frame = odb.steps['LoadStep'].frames[-1]
    disp = last_frame.fieldOutputs['U']
    u2_vals = [v.data[1] for v in disp.values]
    print(f'Job = {job_name}, Max U2 = {max(u2_vals):.6e}')
    odb.close()
```
### 4.8 Example: Full Postprocessing Script

This script opens an existing model and prints it to a `.csv` file.

```python
from odbAccess import openOdb
from abaqusConstants import *

odb = openOdb('Job-1.odb')
step = odb.steps['Step-1']
frame = step.frames[-1]
disp = frame.fieldOutputs['U']
stress = frame.fieldOutputs['S']

with open('results_summary.csv', 'w') as f:
    f.write('Node,U1,U2,Mises\n')
    for d, s in zip(disp.values, stress.values):
        label = d.nodeLabel
        u1, u2, _ = d.data
        mises = s.mises
        f.write(f'{label},{u1},{u2},{mises}\n')

odb.close()
```

