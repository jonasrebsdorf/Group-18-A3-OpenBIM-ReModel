# A3: OpenBIM ReModel

##### Group 18 - s203740, s20

## Use case

When upgrading from IFC2x3 to IFC4, problems can occur with transfering information stored in the different building parts and elements between the two file formats. The problems can occur at different stages of the upgrade and for different reasons depending on the chosen upgrade-method. 


This for example includes incorrect selection of settings during an export from Revit to IFC4, and it is expected that other errors can easily occur with other upgrade methods.

##### Ideal use case

It is therefore relevant to be able to identify potential problems, errors and shortcomings in the upgrade from IFC2x3 to IFC4. It is also important to be able to transfer these missing information from IFC2x3 to IFC4 in the event that problems have occurred. This is clarified in the BPMN-diagram below:

###### BPMN ideal use case

![Alt text](name of SVG file)
<img src=" img/name_of_svg_file.svg ">

##### Current use case

<img src="file:///C:/Users/Jonas/AppData/Roaming/marktext/images/2023-11-07-14-33-14-image.png" title="" alt="" width="604">

Our script provides an overview of discrepancies between the IFC2x3 and IFC4 file. This is done by providing an overview of the amount of elements, so one easily can see the differences between the two file types. At the same time, the script addresses one of the many potential challenges that could arise in the upgrade process, namely the addition of material layer thicknesses in walls, as has been the case with the available SkyLab models.

###### BPMN current use case


The mindset of transferring information from IFC2x3 to IFC4 in the script is assumed to be applicable to other issues, since the script follows three easily adaptable steps:

1. Identify discrepancies between IFC2x3 and IFC4 format

2. Remove/add elements/information from 2x3 to 4

3. Overwrite to new ifc4 file

In our specific case, these steps can be described as follows:

1. A problem with missing thicknesses for materials in the walls is noticed from the generated overview

2. The mat

<mark> GENERAL SCRIPT METHOD.</mark>



#### Documentation of script working

Material information in blender **before** script is used:

![](C:\Users\Jonas\AppData\Roaming\marktext\images\2023-11-07-14-29-29-image.png)

<img src="file:///C:/Users/Jonas/AppData/Roaming/marktext/images/2023-11-07-14-30-10-image.png" title="" alt="" width="609">Material information in blender **after** script is used:









