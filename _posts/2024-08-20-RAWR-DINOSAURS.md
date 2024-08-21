---
title: Dinosaur Prints
categories: [Additive Manufacturing]
tags: [Additive Manufacturing, Showcase]
render_with_liquid: false
---

Most of my prints originate from engineering models, so I wanted to extend into something else. Being a big fan of pre-historic life, I found several 3D scans of fossils, which I then gathered, repaired, and printed. These were all done on FDM machines.

## Utahraptor
The first scan I found, repaired, and printed was the skull of Utahraptor. The final skull is 81 cm long and was printed in sections. 

<img src="pictures/dinosaurs/Utahraptor.JPG" alt="Utahraptor"/>

### Info
Utahraptor, large dinosaur from the Early Cretaceous period, about 135 to 130 million years ago, found in what is now the United States. It is the largest known member of the dromaeosaurid family, and adults measured around 20 to 23 feet long and typically weighed around 1,100 pounds.

## Velociraptor
Velociraptor is a small relative of Utahraptor, and I also printed its skull. The cylinder in the bottle right corner exists to strengthen the jaws.

<img src="pictures/dinosaurs/Velociraptor.JPG" alt="Velociraptor" width="200"/>

### Info
Velociraptor, is a small dinosaur that lived in what is now Asia during the Late Cretaceous period, around 75 to 71 million years ago. It is well-known thanks to its portrayal in the Jurassic Park movies, though it was much smaller in real life, measuring about 5 to 6.8 feet long and weighing between 31 and 43 pounds. 


## Phosphatodraco
I planned on printing the entire skeleton of Utahraptor, but estimates showed indicated that it would be prohibitively expensive. Due to the their (relatively) thin bones, I settled on pterosaurs as a good candidate in its place. Here is the picture of its skull:

<img src="pictures/dinosaurs/Phosphatodraco.JPG" alt="Phosphatodraco Skull"/>

The visible sections are the result of the high humidity breaking the caulking. 

### Method
I acquired a scan of Phosphatodraco which hand to be repaired, remeshed, and posed. I used Blender to do the bulk of the work, and later Meshlab for optimizations. Originally, the skeleton needed eight spools of material to complete. To reduce the material cost, I used my [slicer <span class="fa-solid fa-link"/>](https://asmonta.github.io/posts/Slicer/) to optimize the transformations for the bones, and introduce divides as a means to reduce the overall material use. After optimization, the skeleton needed just under five spools of material to print its ~100 bones.

### Info
Phosphatodraco is a type of pterosaur that lived during the Late Cretaceous period in what is now Northern Africa. Compared to other pterosaurs fossils, it is unique for having a relatively complete neck in the fossil record, though there is some debate over the exact order of these neck bones. This affects estimates of its wingspan, with current adult estimates in the range of 4-5 meters. While still debated, the group Phosphatodraco belongs to, azhdarchids, were originally thought to catch prey from water, but it is now suggested they may have foraged on land like storks. This is what inspired me to pose mine in the standing position. Phosphatodraco survived until a large astroid caused a little bit of a tussle 66 million years ago.

### Relative Size
The bounding box of the final print is 2x2x1.2 meters:

<img src="pictures/dinosaurs/pho_domain_A4.PNG" alt="Relative Size"/>