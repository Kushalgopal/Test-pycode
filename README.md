# Test-pycode

## Code Conversion from JPEG to TIF
```python
from PIL import Image
from osgeo import gdal

# Open the input JPEG image and extract the geo-coordinates
with Image.open('input.jpg') as img:
    lat, lon = img.info['lat'], img.info['lon']
    # Alternatively, you can extract the geo-coordinates using the Exif tags

# Open the input JPEG image using GDAL
src_ds = gdal.Open('input.jpg')

# Create the output TIFF image using GDAL
driver = gdal.GetDriverByName('GTiff')
dst_ds = driver.Create('output.tif', src_ds.RasterXSize, src_ds.RasterYSize, src_ds.RasterCount, src_ds.GetRasterBand(1).DataType)

# Set the geo-coordinates of the output TIFF image
dst_ds.SetGeoTransform(src_ds.GetGeoTransform())
dst_ds.SetProjection(src_ds.GetProjection())

# Copy the data from the input JPEG image to the output TIFF image
for i in range(1, src_ds.RasterCount + 1):
    band = src_ds.GetRasterBand(i)
    dst_band = dst_ds.GetRasterBand(i)
    dst_band.WriteArray(band.ReadAsArray())

# Close the input and output files
src_ds = None
dst_ds = None

```
**NOTE:**
This code opens the input JPEG image using the PIL library and extracts the geo-coordinates. It then opens the same image again using the GDAL library and creates an output TIFF image using the same dimensions and data type as the input image. The geo-coordinates of the output image are set to be the same as the input image using the SetGeoTransform() and SetProjection() methods. Finally, the data is copied from the input image to the output image band by band using a loop.

Note that the above code assumes that the input JPEG image has geo-coordinates embedded in the image metadata. If this is not the case, you may need to extract the geo-coordinates from other sources, such as a separate metadata file. Additionally, you may need to adjust the code to handle other aspects of the image, such as the color profile or compression settings, depending on your specific use case.


**Using Rasterio library from Python to convert to GeoTiff**
```python

# pip install rasterio

import rasterio

# Open the JPEG file
with rasterio.open("input_image.jpg") as src:

    # Prepare the metadata for the output GeoTIFF file
    metadata = src.meta.copy()
    metadata.update({
        "driver": "GTiff",
        "dtype": rasterio.uint8,
        "count": 3 # set to 3 for RGB images
    })

    # Create the output GeoTIFF file
    with rasterio.open("output_image.tif", "w", **metadata) as dst:

        # Copy the JPEG data to the GeoTIFF file
        dst.write(src.read())


```

**NOTE:**
In the above example, the rasterio.open() function is used to open the JPEG file and retrieve its metadata, such as dimensions and spatial reference system. Then, a new GeoTIFF file is created using the same metadata and the rasterio.open() function with the "w" mode. Finally, the rasterio.write() function is used to copy the JPEG data to the GeoTIFF file.

Note that this example assumes that the input JPEG file is an RGB image with 8-bit depth. You may need to adjust the code if your JPEG file has a different format or depth. Also, you may need to specify the spatial reference system explicitly if it's not included in the metadata of the JPEG file.
