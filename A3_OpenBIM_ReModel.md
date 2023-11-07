# A3: OpenBIM ReModel

##### Group 18  -  s203740, s203712

## Use case

When upgrading from IFC2x3 to IFC4, problems can occur with transfering information stored in the different building parts and elements between the two file formats. The problems can occur at different stages of the upgrade and for different reasons depending on the chosen upgrade-method. 


This for example includes incorrect selection of settings during an export from Revit to IFC4, and it is expected that other errors can easily occur with other upgrade methods.

##### Ideal use case

It is therefore relevant to be able to identify potential problems, errors and shortcomings in the upgrade from IFC2x3 to IFC4. It is also important to be able to transfer these missing information from IFC2x3 to IFC4 in the event that problems have occurred. This is clarified in the BPMN-diagram below:

###### BPMN ideal use case

![Alt text](C:\Users\Jonas\Downloads\Ideal_use_case.svg)

##### Current use case

<img title="" src="file:///C:/Users/Jonas/AppData/Roaming/marktext/images/2023-11-07-14-33-14-image.png" alt="" width="604" data-align="center">

Our script provides an overview of discrepancies between the IFC2x3 and IFC4 file. This is done by providing an overview of the amount of elements, so one easily can see the differences between the two file types. At the same time, the script addresses one of the many potential challenges that could arise in the upgrade process, namely the addition of material layer thicknesses in walls, as has been the case with the available SkyLab models.

###### BPMN current use case

![Alt text](C:\Users\Jonas\Downloads\Current_use_case.svg)The mindset of transferring information from IFC2x3 to IFC4 in the script is assumed to be applicable to other issues, since the script follows three easily adaptable steps:

1. Identify discrepancies between IFC2x3 and IFC4 format

2. Remove/add elements/information from 2x3 to 4

3. Overwrite to new ifc4 file

In our specific case, these steps can be described as follows:

1. A problem with missing layer thicknesses for materials in the walls is noticed from the generated overview.

2. For every wall in the IFC4-file, a matching walltype (and its imbedded properties) from the IFC2x3-file is identified and its information is transferred from the IFC2x3-file to the IFC4-file.

3. The materials and their thicknesses from the now updated IFC4-file is identified.

4. A new remodelled IFC4-file is created with the material layer thicknesses from the IFC2x3-file.

## Script

```python
import ifcopenshell
import ifcopenshell.api
from ifcopenshell.util import element
print(ifcopenshell.version + "\n")

ida2x3 = r'C:\Users\dkbma\OneDrive\Skrivebord\Dokumenter\DTU\Kandidat\1 semester\BIM (41934)\A3\data\LLYN-ARK_2x3.ifc'
ida4 = r'C:\Users\dkbma\OneDrive\Skrivebord\Dokumenter\DTU\Kandidat\1 semester\BIM (41934)\A3\data\LLYN-ARK_4.ifc'
# jonas2x3 = r'C:\Users\Jonas\Desktop\Architectural Engineering\1. semester\41934 - Advanced Building Inforrmation Modeling\AdvBIM\data\LLYN - ARK - Space.ifc'
# jonas4 = r'C:\Users\Jonas\Downloads\LLYN - ARK (2).ifc'

#Insert the path to our IFC2x3-file
model2x3 = ifcopenshell.open(ida2x3)

#Insert the path to our IFC4-file
model4 = ifcopenshell.open(ida4)

# Before creating a redesign of the IFC 4 file. The difference between the two files will be investigated.

# Function to count walls in an IFC files to see if some elements was not transferred correctly.
def count_walls(ifc_file):
    model = ifcopenshell.open(ifc_file)
    walls = model.by_type('IfcWall')

    return len(walls)

#Count walls in both IFC files
count2x3 = count_walls(ida2x3)
count4 = count_walls(ida4)

# Print the results
print(f"Number of walls in IFC2x3 file: {count2x3}")
print(f"Number of walls in IFC4 file: {count4}")


def main():
    walls2x3 = load_walls(model2x3)
    walls4 = load_walls(model4)

    new_walls4 = match_layers(walls2x3, walls4)

    new_file_path = r'C:\Users\dkbma\OneDrive\Skrivebord\Dokumenter\DTU\Kandidat\1 semester\BIM (41934)\A3\outputs\modified_model4.ifc'

    print("New walls in ifc4:")
    for key in new_walls4:
        print(f'\n{key} :')
        for layer in new_walls4[key]["layers"]:
            print(f'{" " : <4}{layer} : {new_walls4[key]["layers"][layer][0]: <35} | {str(new_walls4[key]["layers"][layer][1])+"mm" : <8}')

    update_model_and_write(model4, new_walls4, new_file_path)
    print("\nSuccessfully wrote walls to new file!")

# Match layers fra ifc 4 to ifc 2x3 file
def match_layers(source, target):
    for wall4 in target:
        for wall2x3 in source:
            if source[wall2x3]["layers"] is None:
                continue
            if wall4 == wall2x3:
                for layer4 in target[wall4]["layers"]:
                    for layer2x3 in source[wall2x3]["layers"]:
                        if(target[wall4]["layers"][layer4][0] == source[wall2x3]["layers"][layer2x3][0]):
                            target[wall4]["layers"][layer4] = source[wall2x3]["layers"][layer2x3]
    return target


#Load walls from ifc
def load_walls(model):
    walls = model.by_type('IfcWall')
    wall_dict = {}
    for wall in walls:
        wall_name = wall.get_info()["Name"]
        # area = getArea(wall)

        layers = {}
        material, thickness = get_ifc_materials(wall)
        if(len(material) == 0):
            layers = None
        else:
            for i, layer in enumerate(zip(material, thickness)):
                layers["layer"+str(i)] = layer
        wall_dict[wall_name] = {"layers": layers, "product": wall}
    return wall_dict


# Load materials from ifc4 file
def get_ifc_materials(ifc_product):
    material_name = []
    material_thickness = []

    if ifc_product:
        ifc_material = ifcopenshell.util.element.get_material(ifc_product)
        if ifc_material:
            if ifc_material.is_a('IfcMaterial'):
                material_name.append(ifc_material.Name)
                material_thickness.append(0.0) 

            if ifc_material.is_a('IfcMaterialList'):
                for materials in ifc_material.Materials:
                    material_name.append(materials.Name)
                    material_thickness.append(0.0)

            if ifc_material.is_a('IfcMaterialConstituentSet'):
                for material_constituents in ifc_material.MaterialConstituents:
                    material_name.append(material_constituents.Material.Name)
                    material_thickness.append(0.0)

            if ifc_material.is_a('IfcMaterialLayerSetUsage'):
                # Check if 'MaterialLayers' is available
                if hasattr(ifc_material.ForLayerSet, 'MaterialLayers'):
                    for material_layer in ifc_material.ForLayerSet.MaterialLayers:
                        material_name.append(material_layer.Material.Name)
                        material_thickness.append(material_layer.LayerThickness)
                    
            if ifc_material.is_a('IfcMaterialProfileSetUsage'):
                for material_profile in ifc_material.ForProfileSet.MaterialProfiles:
                    material_name.append(material_profile.Material.Name)
                    material_thickness.append(0.0)

    return material_name, material_thickness

# Update new ifc file
def update_model_and_write(model, walls, target_path):
    for wall_name in walls:
        material_set = ifcopenshell.api.run("material.add_material_set", model, name=wall_name, set_type="IfcMaterialLayerSet")
        product = walls[wall_name]["product"]
        for layer_key in walls[wall_name]["layers"]:
            layer_name = walls[wall_name]["layers"][layer_key][0]
            layer_thickness = walls[wall_name]["layers"][layer_key][1]

            layer_material = ifcopenshell.api.run("material.add_material", model, name=layer_name, category=layer_name)
            layer_to_add = ifcopenshell.api.run("material.add_layer", model, layer_set=material_set, material=layer_material)
            ifcopenshell.api.run("material.edit_layer", model, layer=layer_to_add, attributes={"LayerThickness": layer_thickness})
        
        ifcopenshell.api.run("material.assign_material", model, product=product, material=material_set)

    model4.write(target_path)

if _name_ == "_main_":
    main()
```

#### Documentation of script working

Material information in blender **before** script is used:

![](C:\Users\Jonas\AppData\Roaming\marktext\images\2023-11-07-15-12-07-image.png)

![](C:\Users\Jonas\AppData\Roaming\marktext\images\2023-11-07-15-12-50-image.png)

Material information in blender **after** script is used:

<img src="https://scontent-arn2-1.xx.fbcdn.net/v/t1.15752-9/385520761_338475392108483_4848204249677253634_n.png?_nc_cat=104&ccb=1-7&_nc_sid=8cd0a2&_nc_ohc=WA_G9vV_ebQAX-Bztrk&_nc_ht=scontent-arn2-1.xx&oh=03_AdSoSL9LRy0Ph2vho6YQD0FVs5a8FUSIY1_VumPQnsmqPw&oe=6571B709" title="" alt="Ingen tilgængelig beskrivelse." width="648">

![Ingen tilgængelig beskrivelse.](https://scontent-arn2-1.xx.fbcdn.net/v/t1.15752-9/371480522_1812465755844437_1436028577262163088_n.png?_nc_cat=104&ccb=1-7&_nc_sid=8cd0a2&_nc_ohc=l61DBT_uPu4AX8t0JaK&_nc_ht=scontent-arn2-1.xx&oh=03_AdRel28SR1RVNclvfSVdrSQY83xxzi805f5DVmpyifGiRg&oe=6571B6E3)







