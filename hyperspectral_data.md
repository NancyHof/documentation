# Hyperspectral Data

## Calibration and Correction

This is a draft algorithm to retrieve spectral reflectance based on my understanding of the (possibly soon-to-be) available inputs. Suggestions and correction welcome (the more specific, the better). For simplicity, the algorithm description currently omits the time dimension. It is implicit in all quantities below except rfl_wht.

### Inputs and Outputs 

syntax: `Variable(dimensions) [units]`
$$
I_p(\lambda,y,x) = F_{\lambda}(\lambda) R_p(\lambda) C(\lambda)
$$

#### Inputs: Required known or measured inputs:

1. `uint16  xps_img(wavelength,y,x) [xps] = `Exposure from experiment image (i.e., plants) (known, VNIR, SWIR sensors)
2. `uint16  xps_wht(wavelength,x) [xps]   = `Exposure from white reference sheet/panel (measured by VNIR, SWIR sampling period? location?)
3. `float32 rfl_wht(wavelength) [fraction]      = `Reflectance of white reference (factory calibration) (assume time-constant?)
4. `float32 flx_dwn(wavelength) [W m-2 um-1]    = `Downwelling spectral irradiance (measured by environmental sensor. units?)

#### Intermediate derived quantities:

1. `float32 cst_cnv(wavelength) [xps/(W m-2 um-1)] = `Proportionality constant between reflected spectral flux and Exposure (derived)
2. `float32 flx_upw(wavelength) [W m-2 um-1]                    = `Upwelling  spectral flux (derived. and possibly measured for closure?)

#### Outputs

2. `float32 rfl_img(wavelength,y,x) [fraction]     = `Reflectance of image (i.e., plants)

### Proposed Algorithm to retrieve reflectance from measurements:

1. Assume image exposure linear with incident spectral flux, this implies
  * `xps_wht=flx_dwn*rfl_wht*cst_cnv`
2. Derive proportionality constant from calibration:
  * `cst_cnv=xps_wht/(flx_dwn*rfl_wht)`
3. Assume proportionality constant for calibration sheet and plant image are identical, this implies
  * `xps_img=flx_dwn*rfl_img*cst_cnv`
4. Derive plant reflectance from exposure
  * `rfl_img=xps_img/(flx_dwn*cst_cnv)`
5. (Optional) Derive upwelling spectral flux from reflectance and compare it to measured upwelling spectral flux (if available) for closure/validation
  * `flx_upw=flx_dwn*rfl_img`
6. (Optional) Apply PAR-sensor SRF to downwelling irradiance, integrate, compare to measured PAR for closure (integration would require detailed information or assumptions about bandpass of each spectral channel).
7. More?

Before implementing this, I would like feedback and/or sign-off by @dlebauer @nfahlgren @max-zilla @pless @solmazhajmohammadi and @LTBen. 


### Next steps

More assumptions, input measurements, and/or more sophisticated algorithms would incorporate these additional sources of information:

1. BRDF (angular reflectance properties) of plant/leaves
2. 3D orientation of reflective surfaces (plant/leaves)
3. BRDF (angular reflectance) of calibration sheet
4. Direct/diffuse partitioning of downwelling flux


### Notes

* Units of exposure are vague. Exposure is similar to a photon counter modulated by the spectral response function (SRF) of the sensor. The units are denoted [xps] and range from [0..2^16-1] = [0..65535]. 
* add QAQC tests for sensitivity and saturation in each band
* cross validate with other sensors with know spectral response functions.
* NB: Three required inputs (xps_wht, rfl_wht, flx_dwn) are not ready to use. Their location, units, and/or sampling intervals are unknown. Only xps_img is ready. Please tell me how to get the other three in the notes below. (@LTBen, @markus-radermacher-lemnatec)

## Hyperspectral Data Formats

This is a proposal for spectral and imaging data to be provided as HDF-5 / NetCDF-4 data cubes for computing and downloading by end users.

Following [CF naming conventions](http://cfconventions.org/Data/cf-standard-names/29/build/cf-standard-name-table.html), these would be in a netcdf-4 compatible / well behaved HDF format. 

### Radiance data

#### Variables

| variable name | units | dim 1 | dim 2 | dim 3 | dim 4 | dim 5 |
|----|----|----|----|----|----|----|----|
| surface_bidirectional_reflectance   |  0-1 |  lat   | lon   | time   |  radiation_wavelength |  |
| bandwidth |  0-1 |  lat   | lon   | time   |  radiation_wavelength |  |
| upwelling_spectral_radiance_in_air | W m-2 m-1 sr-1 |  lat   | lon   | time   |  radiation_wavelength | zenith_angle |

note: upwelling_spectral_radiance_in_air may only be an intermediate product (and perhaps isn't exported from some sensors?) so the focus is really on the reflectance as a Level 2 product.


#### Dimensions 

| dimension | units |  notes |
|----|----|---| 
| time | hours since 2016-01-01|  first dimension |
| latitude | degrees_north |   (or alt. projection_y_coordinate) | 
| longitude | degrees_east |  (or alt. porjection_x_coordinate below)|
| _projection_x_coordinate_ | m | can be mapped to lat/lon with grid_mapping attribute |
| _projection_y_coordinate_ | m |   can be mapped to lat/lon with grid_mapping attribute | 
| radiation_wavelength | m  |
| zenith_angle | degrees |
|  _optional_  |    |
|  sensor_zenith_angle | degrees |
|  platform_zenith_angle | degrees  |  
