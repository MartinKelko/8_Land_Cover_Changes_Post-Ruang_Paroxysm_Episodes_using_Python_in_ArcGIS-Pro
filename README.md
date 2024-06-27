# Ruang Volcano Vegetation Change Timelapse

## Description

This script visualizes the dramatic vegetation changes and subsequent regrowth following the Indonesian Ruang volcano's two paroxysm phases in April. Utilizing the Python Notebook within the ArcGIS Pro environment, the script processes a series of GeoTIFF images to create a GIF timelapse of the vegetation changes.
https://medium.com/@martin2kelko/visualizing-the-impact-land-cover-changes-post-ruang-paroxysm-episodes-using-python-in-arcgis-pro-db898aba6c76?source=your_stories_page-------------------------------------

## How to Use

### Prerequisites

- ArcGIS Pro installed with an active license.
- Python environment configured with `arcpy` and `imageio` libraries.
- GeoTIFF images representing different time periods of vegetation cover.
- Ensure the images and output directory paths are correctly set in the script.

### Script Explanation

1. **Import Libraries**

   Import the necessary libraries for the task.

   ```python
   import arcpy
   import os
   import imageio.v2 as imageio
   import shutil

arcpy.env.overwriteOutput = True

image_files = [
    "C:/EsriTraining/Ruang_land_cover_change/ruang_0318.tif",
    "C:/EsriTraining/Ruang_land_cover_change/ruang_0427.tif",
    "C:/EsriTraining/Ruang_land_cover_change/ruang_0512.tif",
    "C:/EsriTraining/Ruang_land_cover_change/ruang_0527.tif",
    "C:/EsriTraining/Ruang_land_cover_change/ruang_0530.tif"
]

output_directory = "C:/EsriTraining/Ruang_land_cover_change/ruang_output_directory"

if not os.path.exists(output_directory):
    os.makedirs(output_directory)

def clear_output_directory(directory):
    for filename in os.listdir(directory):
        file_path = os.path.join(directory, filename)
        try:
            if os.path.isfile(file_path) or os.path.islink(file_path):
                os.unlink(file_path)
            elif os.path.isdir(file_path):
                shutil.rmtree(file_path)
        except Exception as e:
            print(f"Failed to delete {file_path}. Reason: {e}")

clear_output_directory(output_directory)

try:
    aprx = arcpy.mp.ArcGISProject("CURRENT")
    map = aprx.listMaps()[0]
except Exception as e:
    print("Failed to load the project or map. Reason:", e)
    quit()

def update_layer(image_path):
    for lyr in map.listLayers():
        map.removeLayer(lyr)
    
    new_layer = arcpy.management.MakeRasterLayer(image_path, "temp_layer")
    map.addLayer(new_layer.getOutput(0))

frame_count = 0
for image_path in image_files:
    update_layer(image_path)
    
    map_view = aprx.activeView
    if hasattr(map_view, 'exportToPNG'):
        layer_extent = arcpy.Describe(map.listLayers()[0]).extent
        map_view.camera.setExtent(layer_extent)
        
        width = layer_extent.width
        height = layer_extent.height
        
        frame_path = os.path.join(output_directory, f"ruang_{frame_count:03d}.png")
        map_view.exportToPNG(frame_path, width=int(width), height=int(height), resolution=600)
        frame_count += 1
    else:
        print("Map view is not available.")

print("Frames exported successfully.")

frame_images = [os.path.join(output_directory, f) for f in sorted(os.listdir(output_directory)) if f.startswith('ruang_') and f.endswith('.png')]

gif_path = os.path.join(output_directory, "timelapse.gif")
with imageio.get_writer(gif_path, mode='I', fps=1) as writer:
    for _ in range(5):
        for frame_image in frame_images:
            image = imageio.imread(frame_image)
            writer.append_data(image)

print("Time-lapse GIF created successfully.")

python create_timelapse.py
