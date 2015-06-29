# File format testing

#### Formats:
* HDF5
* netCDF4
* GeoTIff
* ENVI
* JPEG2000

#### Compression
* HDF5:		gzip/decompress
* netCDF4:	gzip/decompress
* GeoTiff:	LZW
* ENVI:		None, gzip/decompress (compare raw vs compressed)
* JPEG2000:	Wavelets (lossless)

#### Python Api:
* HDF5:		h5py
* netCDF4:	netcdf4
* GeoTiff:	GDAL, rasterio
* ENVI:		GDAL (for uncompressed), gzip for compressed
* JPEG2000:	GDAL, rasterio

#### Read tests
Each read that occurs will be from a randomly generated x,y location.
* Single pixel over 3D domain
* 2D spatial block
* 2D spatial block over 3D domain (returns a 3D array)

For a spatial block, the randomly chosen x,y location can be used as the UL
starting poing for teh rectangular block. There is pre-existing code in the
gaip package for generating random x,y locations. The code can be adapted for
the spatial read tests to ensure that the spatial blocks won't go out of
bounds.

The design behind reading random locations is to minimise and potential
advantages the api's may receive from caching previous results of read tests.
It may still be required to force each api to flush the cache.

#### Algorithm test
Each file format will perform a test against a specific algorithm/workflow
that is designed to read several group datasets, perform several calculations
and output the result. The bare soils workflow is a candidate for spatial and
z-axis processing. WOfS is a candidate for processing either purely spatial
blocks or a combination of spatial/z-axis. The chunk/block size defined for
the array during creation will be used for the processing chunk size.

#### File structure
The files used for testing will make use of existing data for a given cell
across all of time. The data will consist of the following groups; NBAR,
Fractional Cover, and Pixel Quality. The underlying structure will contain 3D
arrays for each of datasets (eg Blue, NIR, Photosynthetic, Bare Soil)
contained aforementioned groups.
Queries can be structured to also return more than one dataset, in either a 4D
array or separate 3D arrays.
The HDF5, netCDF4 files will be defined as 3D chunks, while the GeoTiff file
will do an equivalent 2D chunk. For example (1, 100, 4000) is a 3D chunk, and
the equivalent 2D is (100, 4000).
Each read will need to conform to a band sequential memory layout, i.e.
(z, y, x).
Separate files for each group dataset will be generated for each of the file
formats. Additionally, the HDF5 and netCDF4 file formats will also have a
single file containing all groupd datasets. Code from the *tesserae* repository
will be re-used in this instance, with some minor work involved to create a
netCDF4 equivalent.
HDF5 and netCDF4 will attempt to have chunk sizes as close as possible to the
cache size of 1CPU on the raijin cluster at NCI.

#### Interleaving
Related to the _file structure_ topic, the files will be interleaved for each
case that the relevant api allows.

* BSQ:		Band Interleaved (z, y, x)
* BIL:		Band Interleaved by Line (y, z, x)
* BSQ:		Band Interleaved by Pixel (y, x, z)

Due to the requirement for (z, y, x) memory layout upon return by the api,
HDF5 and NetCDF4 will mimic a BIL product.

* HDF5:		BSQ/BIL
* netCDF4:	BSQ/BIL
* GeoTiff:	BSQ/BIP (band/pixel according to the libtiff documentation)
* ENVI:		BSQ/BIL/BIP
* JPEG2000:	Library default

#### Requirements
All files must be able to be created in tiled/chunk fashion, i.e. not require
the entire (z, y, x) image in memory for it to then be written to disk.
For consistancy, the memory layout of (z, y, x) should be preserved in the
best interests of aligning with the OGC GRID standard. This aids in not only
the display of imagery, but also to developers who then know how to structure
code according to the memory layout.
