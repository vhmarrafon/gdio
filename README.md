
# GDIO - Gridded Data IO

A simple and concise gridded data IO library for reading multiples grib and netcdf files, automatic spatial interpolation of the all data to a single resolution.

The library gdio is based on my own professionals and personal needs as a meteorologist. 
The currents libraries always fail when you need to read handle multiples large 
netcdf/grib files, with differents resolutions and timesteps.

After version 0.1.2 the output data was converted to object with key-values accessible using attribute notation, and after version 0.1.8 a new multilevel dictionary data structure.

## Instalation
```
conda config --env --add channels conda-forge
conda install -c rodri90y gdio

if you are using pip install, before install manually the requirements

conda create -n envname --file requirements.txt
pip install gdio

```

#### Required dependencies

conda config --add channels conda-forge

+ Python (3.7.9=> or later)
+ numpy (1.18.4 or later)
+ netCDF4 (1.5.1.2 or later)
+ eccodes (2.12.0 or later)
+ python-eccodes
+ pyproj


#### Optional dependencies
+ scipy (1.4.1 or later)

#### Testing
```
python -m unittest 
```


## Reading files
The gdio support the IO of grib1/2 and netcdf file formats, allowing the time and spatial subdomains cut.

This library unifies categories of information (variable, level, members) in a single 
data structure as a multilevel dictionary/attribute, regardless of the format read (netcdf and grib), the 
output format will be standardized in order to simplify access to the data.

In the dataset first level the following parameters are accessible: ref_time, time_units and time in addition to the weather variables.
ds.ref_time, ds.time
At the variable level we have: level_type, param_id, long_name, parameterUnits, latitude and longitude and at vertical level (isobaricInh, surface, etc) the variable data as value and level are exposed.

Structure data:

    + dataset
        + ref_time
        + time_units
        + time
        + variable (u,v,2t,etc) 
            + level_type
            + param_id
            + long_name
            + parameter_units
            + latitude
            + longitude
            + isobaricInhPa/surface/maxWind/sigma (any level type key)
                + value
                + level
                + members

Example:
            
    ds.time
    ds.time_units
    ds.v.latitude
    ds.v.isobaricInhPa.value
    ds.v.isobaricInhPa.level
    ds.v.isobaricInhPa.members


#### Netcdf
The class netcdf encapsulates all netcdf functions of reading and writing, as well as the cutting of time and spatial domains, returning the netcdf data as a dictionary type. The returned dictionary contains for each variable the value, param_id, type_level, level and parameter_units property.

Simple reading
```
from gdio.netcdf import netcdf
nc = netcdf(verbose=False)

ds = nc.nc_load('tests/data/era5_20191227_lev.nc')
>>> ds.keys()
dict_keys(['ref_time', 'time_units', 'time', 'r', 't', 'u', 'v'])
>>> print(ds.u.isobaricInhPa.value.shape)
(1, 2, 7, 161, 241)
>>> print(ds.u.level_type)
['isobaricInhPa']
>>> print(ds.u.keys())
dict_keys(['isobaricInhPa', 'param_id', 'long_name', 'parameter_units', 'latitude', 'longitude', 'level_type'])
>>> print(ds.u.isobaricInhPa.level)
[200, 300, 500, 700, 800, 950, 1000]
>>> print(ds.u.parameter_units)
m s**-1
>>> print(ds.u.param_id)
None
```

Reading a subsample in time (time 12-24) and space (bbox -30,-60 and 10,-40). The returned multilevels dictionary/attributes contains for each variable the value, param_id, type_level, level and parameter_units property.

```
ds = nc.nc_load('data/era5_20191227_lev.nc', cut_domain=(-30, -60, 10, -40), cut_time=(12, 24))
>>> print(ds.u.isobaricInhPa.value.shape)
(1, 1, 7, 80, 40)
```
Rename variables
A dictionary input will rename variables names (key) for a new name (value).
Eg. {'tmpmdl': 't', 'tmpprs': 't'}

```
ds = nc.nc_load('data/era5_20191227_lev.nc', rename_vars={'u':'10u'})
>>> ds.keys()
dict_keys(['ref_time', 'time_units', 'time', 't', '10u', 'v', 'r'])
```

Writing a netcdf file

From the loaded dataset
```
nc.nc_write('data/output.nc', ds)
```
From a dictionary
```
from datetime import datetime
import numpy as np
from gdio.netcdf import netcdf

nc = netcdf(verbose=False)

ds = {'ref_time': datetime(2019, 12, 27, 0, 0), 
      'time_units': 'hours', 
      'time': np.array([12]),
      'u': {'isobaricInhPa': {  'value': np.random.random((1, 7, 80, 40)),
                                'level': [200, 300, 500, 700, 800, 950, 1000]
                              },
            'param_id': None, 
            'long_name': 'U component of wind', 
            'level_type': ['isobaricInhPa'],
            'parameter_units': 'm s**-1',
            'longitude': np.array([300. , 300.5, 301. , 301.5, 302. , 302.5, 303. , 303.5,
               304. , 304.5, 305. , 305.5, 306. , 306.5, 307. , 307.5,
               308. , 308.5, 309. , 309.5, 310. , 310.5, 311. , 311.5,
               312. , 312.5, 313. , 313.5, 314. , 314.5, 315. , 315.5,
               316. , 316.5, 317. , 317.5, 318. , 318.5, 319. , 319.5]),
            'latitude': np.array([-30. , -29.5, -29. , -28.5, -28. , -27.5, -27. , -26.5,
               -26. , -25.5, -25. , -24.5, -24. , -23.5, -23. , -22.5,
               -22. , -21.5, -21. , -20.5, -20. , -19.5, -19. , -18.5,
               -18. , -17.5, -17. , -16.5, -16. , -15.5, -15. , -14.5,
               -14. , -13.5, -13. , -12.5, -12. , -11.5, -11. , -10.5,
               -10. ,  -9.5,  -9. ,  -8.5,  -8. ,  -7.5,  -7. ,  -6.5,
                -6. ,  -5.5,  -5. ,  -4.5,  -4. ,  -3.5,  -3. ,  -2.5,
                -2. ,  -1.5,  -1. ,  -0.5,   0. ,   0.5,   1. ,   1.5,
                 2. ,   2.5,   3. ,   3.5,   4. ,   4.5,   5. ,   5.5,
                 6. ,   6.5,   7. ,   7.5,   8. ,   8.5,   9. ,   9.5]),
            }
      }

nc.nc_write('data/output.nc', ds)
```

#### Grib
The class netcdf encapsulates all grib functions, as well as the cutting of time and spatial domains , returning the netcdf data as a dictionary type.

Simple reading
```
from gdio.grib import grib
gr = grib(verbose=False)
ds = gr.gb_load('data/era5_20191226-27_lev.grib')

>>> ds.keys()
dict_keys(['ref_time', 'time_units', 'time', 't', 'u', 'v', 'r'])
>>> print(ds.u.isobaricInhPa.value.shape)
(1, 4, 7, 241, 281)
>>> print(ds.u.level_type)
['isobaricInhPa']
>>> print(ds.u.keys())
dict_keys(['isobaricInhPa', 'param_id', 'long_name', 'parameter_units', 'latitude', 'longitude', 'level_type'])
>>> print(ds.u.isobaricInhPa.level)
[200, 300, 500, 700, 800, 950, 1000]
>>> print(ds.u.parameter_units)
m s**-1
>>> print(ds.u.param_id)
131
```

Reading a subsample in time (time 12-24) and space (bbox -30,-60 and 10,-40)

```
ds = gr.gb_load('data/era5_20191226-27_lev.grib', cut_domain=(-30, -60, 10, -40), cut_time=(12, 24))
```

Setting the ensemble grouping grib id key

```
gr.fields_ensemble = 'perturbationNumber'
gr.fields_ensemble_exception = [0]
```

Filtering by a grib key, dict with grib parameters at form of pair key:
values (list or single values)
eg: filter_by={'perturbationNumber': [0,10],'level': [1000,500,250]}
                            or filter_by={'gridType': 'regular_ll'}

```
ds = gr.gb_load('tests/data/era5_20191226-27_lev.grib', 
                cut_domain=(-30, -60, 10, -40), 
                cut_time=(12, 24), 
                filter_by={'perturbationNumber': 0, 'level':[200,500,950]})
>>> print(ds.u.isobaricInhPa.level)
[200, 500, 950]
```
Rename variables
A dictionary input will rename variables names (key) for a new name (value).
Eg. {'tmpmdl': 't', 'tmpprs': 't'}

```
ds = gr.gb_load('data/era5_20191227_lev.nc', rename_vars={'u':'10u'})
>>> ds.keys()
dict_keys(['ref_time', 'time_units', 'time', 't', '10u', 'v', 'r'])
```
Sorting grib parameter before (warning high, consumption of memory). 
Fix grib files unstructured or non-standard.
```
ds = gr.gb_load('data/era5_20191227_lev.nc', sort_before=True)
```

### Reading multiple files
This class has high level routines for multiple files and type reading, returning the netcdf/grib data as a list of dictionary type.

```
from gdio.core import gdio

ds = gdio(verbose=False)
ds.mload(['tests/data/era5_20191226-27_lev.grib', 'tests/data/era5_20191227_lev.nc'],  
        merge_files=True, uniformize_grid=True, cut_domain=(-30, 300, 10, 320), 
        filter_by={'perturbationNumber': 0}, inplace=True)

>>> ds.dataset[0].keys()
dict_keys(['ref_time', 'time_units', 'time', 'longitude', 'latitude', 't', 'u', 'v', 'r'])
>>> print(ds.dataset[0].u.isobaricInhPa.value.shape)
(1, 6, 7, 160, 80)

```

Setting the ensemble grouping grib id key

```
ds.fields_ensemble = 'perturbationNumber'
ds.fields_ensemble_exception = [0]
```

### Selecting a sub sample in mload dataset
Select data by coordinates (date, latitude, longitude, levels and members)

```
sub_set = ds.sel(dates=[datetime(2019,12,26,12,0)], latitude=[-23.54,-22], longitude=[-46.64,-42.2], level=[2,6])

>>> print(sub_set[0].get('u').isobaricInhPa.value.shape)
(1, 1, 4, 6, 18)
```

## Routines
#### gdio.mload
```
mload(files, vars=None, merge_files=True, cut_time=None,
      cut_domain=None, level_type=None, filter_by={},
      uniformize_grid=True, sort_before=False, inplace=False)
```          
Load multiple files (netcdf/grib) returning the data as a list of dictionary type interpolating the data to a same grid

**files:               list**

files names
                                    
**uniformize_grid:     boolean**\
interpolate all ncs to first nc grid specification

**vars:                list**\
variables names

**merge_files:         boolean**\
merge files

**cut_time:            tuple**\
                        range of time to cut ex.: (0,10)/(0,None)/(None,10)

**cut_domain:          tuple**\
                        range of latitudes and longitudes to cut: (lat1, lon1, lat2, lon2)
                        ex.: (-45,-90,20,-30)/(-45,None,20,-30)/(None,-90,None,-20)

**level_type:          list**\
                        type of level (hybrid, isobaricInhPa, surface)

**filter_by:           dictonary**\
 dict with grib parameters at form of pair key:values (list or single values)
                        eg: filter_by={'perturbationNumber': [0,10],'level': [1000,500,250]}
                        or filter_by={'gridType': 'regular_ll'}|

**rename_vars:         dictonary
                        rename variables names (key) for a new name (value).
                        Eg. {'tmpmdl': 't', 'tmpprs': 't'}

**sort_before:         bool**\
                        Sort fields before process validityDate, validityTime, paramId, typeOfLevel,
                        perturbationNumber and level. Warning high consumption of memory, just use
                        when the grib data structure is not standard

**return:**                    list of dictionaries


#### gdio.sel
```
sel(data=None, latitude=None, longitude=None, 
    dates=None, level=None, member=None, date_format="%Y-%m-%d %H:%M")
```

Select data by coordinates (date, latitude, longitude, levels and members)

**data:       list of dictionaries**\
                             raw dataset
                             
**latitude:     list of floats**\
                             latitudes
                             range of latitudes to select: [lat1, lat2]
                             especific latitudes (1 or >2) [lat1, lat2, lat3, ...]

**longitude:    list of floats**\
                             range of longitudes to select: [lon1, lon2]
                             especific longitudes (1 or >2) [lon1, lon2, lon3, ...]

**dates:        list of datetime/string**\
                             datetime/string date
                             range of dates to select: [date1, date2]
                             especific dates (1 or >2) [date1, date2, date3, ...]

**level:        list of int**\
                             range of levels to select: [level1, level2]
                             especific levels (1 or >2) [level1, level2, level3, ...]

**member:       list of int**\
                             range of levels to select: [member, member]
                             especific levels (1 or >2) [level1, level2, level3, ...]

**return:**     list of dictionaries


#### gdio.grib.gb_load
```
def gb_load(selfifile, vars=None, level_type=None,
            cut_time=None, cut_domain=None, filter_by={},
            rename_vars={}, sort_before=False)
```
Load grib file

**ifile:       string**\
                            grib 1 or 2 file name

**vars:        list**\
                            variables short name or id parameter number

**cut_time:    tuple**\
                            range of time to cut ex.: (0,10)/(0,None)/(None,10)

**cut_domain:  tuple**\
                            range of latitudes and longitudes to cut: (lat1, lon1, lat2, lon2)
                            ex.: (-45,290,20,330)/(-45,None,20,330)/(None,290,None,320)

**level_type:  list**\
                            type of level (hybrid, isobaricInhPa, surface)

**filter_by:   dictonary**\
                            dict with grib parameters at form of pair key:values (list or single values)
                            eg: filter_by={"perturbationNumber": [0,10],"level": [1000,500,250]}
                            or filter_by={"gridType": "regular_ll"}

**rename_vars: dictonary**\
                            rename variables names (key) for a new name (value).
                            Eg. {"tmpmdl": "t", "tmpprs": "t"}

**sort_before: bool**\
                            Sort fields before process validityDate, validityTime, paramId, typeOfLevel, perturbationNumber and level
                            Warning high consumption of memory, just use when the grib data structure is not standard

**return: dictonary/attributes**\
multiple time data container

#### gdio.netcdf.nc_load
```
nc_load(ifile, vars=None, cut_time=None, cut_domain=None, level_type=None, rename_vars={}):
```

Load netcdf files


**ifile:       string**\
                    netcdf file name
                    
**vars:        list**\
                    variables short name
                    
**cut_time:    tuple**\
                    range of time (absolute) to cut ex.: (0,10)/(0,None)/(None,10)
                    
**cut_domain:  tuple**\
                    range of latitudes and longitudes to cut: (lat1, lon1, lat2, lon2)
                    ex.: (-45,290,20,330)/(-45,None,20,330)/(None,290,None,320)
                    
**level_type:  list**\
                    type of level (hybrid, isobaricInhPa, surface)

**rename_vars: dictonary**\
                            rename variables names (key) for a new name (value).
                            Eg. {"tmpmdl": "t", "tmpprs": "t"}
                            
**return: dictonary/attributes**\
multiple time data container

#### gdio.netcdf.nc_write
```
nc_write(ifile, data, zlib=True, netcdf_format='NETCDF4')
```

Write netcdf file

**ifile:           string**\
                                file path
                                
**data:            dict**\
                                dataset
                                
**zlib:            bool**\
                                enable compression
                                
**netcdf_format:   string**\
                                netcdf format: NETCDF4, NETCDF4_CLASSIC, NETCDF3_CLASSIC or NETCDF3_64BIT

#### gdio.remapbil
```
remapbil(data, lon, lat, lon_new, lat_new, order=1, masked=False)
```

Interpolate data to new domain resolution

**data: array**\
                        3D data (time,lon,lat)

**lon: array**

**lat: array**

**lon_new: array**\
                        new grid logitudes

**lat_new: array**\
                        new grid latitudes

**order:   int**\
                        0- nearest-neighbor, 1 - bilinear, 2 - cubic spline

**masked: boolean**\
                        If True, points outside the range of xin and yin
                        are masked (in a masked array). If masked is set to a number

**return: 3D array**

## Release History


## Meta
Rodrigo Yamamoto codes@rodrigoyamamoto.com

https://github.com/rodri90y/gdio

## Contributing
* 0.1.8.7
    * alpha release
    

## License

MIT
