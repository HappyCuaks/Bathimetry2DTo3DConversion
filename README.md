# Bathymetric Map to 3D Model

This repository contains the process of transforming bathymetric data from a `.laz` file into a 3D model. The process is explained in three main steps, from obtaining the `.laz` file to generating a 3D mesh and preparing it for 3D printing using Blender.

## Table of Contents

1. [Getting the .laz File](#getting-the-laz-file)
2. [Generating an Image from the .laz File](#generating-an-image-from-the-laz-file)
3. [Creating a 3D Mesh from the Image](#creating-a-3d-mesh-from-the-image)
4. [Editing the 3D Model in Blender](#editing-the-3d-model-in-blender)

## Getting the .laz File

Bathymetric data is typically stored in `.laz` files, which are compressed LiDAR data files. Here's how you can obtain a `.laz` file:

1. **Find a Data Source**: Bathymetric data can be obtained from various sources such as the USGS, NOAA, or other geological data repositories. Also inside laz.db databases already created.
2. **Download the File**: Ensure that the data covers the region you're interested in and download the `.laz` file to your local machine.

## Generating an Image from the .laz File

Once you have the `.laz` file, the next step is to generate an image that represents the bathymetric map. For this, we used the program QGIS:

1. **Read the .laz File**: We used QGIS as it is easier to operate. Drag and drop .laz into QGIS window.
2. **Process the Data**: Extract the relevant points from the file and convert them into a grayscale image, where the brightness corresponds to depth.
    1. **Change `.laz` PointCloud to GreyScale**
        1. **Open Layer Settings**: Double left click on the "Layers" window (bottom left).
        2. **Navigate to Attibute Settings**:  Navigate to Simbology>Attribute by Ramp.
        3. **Change Color Ramp**: See option "Color ramp" and select Greys from the drop down menu.
    2. **Convert LAZ layer to raster Layer**: Convert your `.laz` file into a format that QGIS can work with, such as a raster GeoTIFF DEM (Digital Elevation Model).
        1. **Install LASTools Plugin**: Download LASTools Plugin going to Plugins Menu and clicking on "Manage and Install Plugins". Then, search for LASTools plugin
        2. **Install LASTools Library**: QGIS will try to find LASTools library in <C:\\LAStools\\bin\\las2dem>. Make sure that LASTools are installed in your system and located in this directory. LASTools library can be found in: [LASTools Official Website](https://rapidlasso.de/downloads/)
        3. **Convert LAZ CloudPoints into GeoTIFF**:
            1. Find the newly installed Toolbox in Processing menu by clicking "Toolbox". 
            2. Search for las2dem inside LASTools package tool and double click. 
            3. Select your `.laz` file as input and configure as recommended by the IDE on the right hand side of the window. To gain more resolution, use the <step size / pixel size> in order to gain more or less resolution. We used 0.1 and that was enough for our application, but we tried different values in between 0 and 1. 
            4. Set output file by selecting "Save to File" and the file must be given the extension `.tif` so las2dem tool understands that we want a GeoTiff file.
    3. **Create Area of Interest (AoI)**:
        1. **Create new ShapeFile Layer**: Set Geometry Type to "Polygon".
        2. **Enable editing**: Right click on the layer and click on "Toggle Editing" option.
        3. **Enable Shape Digitizing Toolbar**: Go into View Menu > Toolbars and enable Shape Digitizing Toolbar.
        4. **Create Rectangle over AoI**: Use the Shape Digitizing Toolbar to create a rectangle that represents your AoI using "Rectangle from 3 points (distance) option.
        5. **Disable editing**: Once the editing is done, Right click on the layer and click on "Toggle Editing" option. Editing should deactivate.
    4. **Clip GeoTiff DEM layer using AoI**:
        1. **Remove laz layer or disable it**
        2. **Import newly created `.tif` file**
        3. **Make sure Geotiff DEM layer and AoI are enabled**
        4. **Clip the DEM**
            1. Go to Raster menu > Extraction > Clip Raster by Mask Layer
            2. Input Layer should be the previously created DEM layer.
            3. Mask Layer should be the AoI polygon we just created
            4. Set the file it will be saved into. Here the extension is provided by QGIS.
            5. Run the tool. > **Note:** Errors like this [ERROR 1: Did not get any cutline features](https://gis.stackexchange.com/questions/210427/getting-clip-raster-error-1-did-not-get-any-cutline-features) and [ERROR 1: Cannot compute bounding box of cutline. Cannot find source SRS](https://gis.stackexchange.com/questions/32394/rasterfile-clipping-error-cannot-compute-bounding-box-of-cutline) can be found. Use the forum to sor them out.

3. **Save the Image**: Export the processed data as an image file `.tiff`, which will be used in the next step.

## Generating a 3D Mesh (.obj) from a DEM

To do this, we used a free to use SW called Terresculptor.

1. **Import the DEM into Terresculptor**: Go to File menu. Then click in "Import Terrain". A new window will appear and in the different file formats proposed, choose "TIFF/GeoTIF \[new\] (*tif, *tiff)". We will then find our GeoTIFF file with our DEM. Select the file and Open it. Make sure option "Fill Holes" is selected, just in case.
2. **Smooth the Surface**: If topografic lines and other hard edges need to be smoothened out, use this Smooth tool. It can be found inside the Filter menu. Play with parameters "Size", "Strength" and "Passes" to modify the surface of the model.
3. **Export the 3D model**: Inside File menu, Click on "Export Terrain" and choose Alias Wavefront Object (`*.obj`).Then, proceed to Save the file and accept default configuration.

## Curate the 3D mesh to make if 3D print ready

To make the appropiated edits, we used Blender.

1. **Scale the object to an appropiated X and Y**: In our case, for our printer, 25x25cm XY. To do this, select the object and press "N".
2. **Decimate the object**: In the Properties panel (usually located on the right side), go to the Modifiers tab (the wrench icon). Click in Add Modifier and find Generate>Decimate. We chose to decimate less than 25% of the triangles. Pivot between wireframe and shading viewpoint modes to understand better the effect of this modifier and make sure no important details are lost.
3. **Trim the plane**: If the plane generated by  
    1. In our case, we generated a cube to trim the `.obj` to 24x24cm. Leaving 0.5cm per side for tolerance. Mkae sure both objects are aligned to their centers.
    2. In the Properties panel (usually located on the right side), go to the Modifiers tab (the wrench icon). Click in Add Modifier and find Generate>Boolean.
    3. Use Intersect mode and select the Cube as intersecting Object.
4. **Close the plane to form an object with volume**:
    1. Go into Edit Mode (Can press Tab for this).
    2. Press 2 to switch to edge select mode (or click the edge select button in the toolbar).
    3. Press Alt + Shift + Left Click on one of the outer edges of your surface. This should select the entire boundary edge loop around your surface.
    4. Select the 4 sides of the plane.
    5. Press "D" to duplicate the points.
    5. Press "E" to extrude.
    6. With the vertices selected, press S to scale.
    7. Press Z to constrain the scaling to the Z-axis.
    8. Type 0 and press Enter.
    9. Move the newly created vertex points along the "Z" axis and lower them to the lowest vertex of the mesh.
    10. Still having all the newly created vertex selected, press "F" for Blender to create a new face connecting them all and closing the 3D mesh. 
5. **Export the new Mesh to .stl**: This can be done by going into File menu and clicking in Export>STL (.stl) option. Other formats can be exported depending on what you need to work with. We used .stl because we wanted to 3D print this specific area. 