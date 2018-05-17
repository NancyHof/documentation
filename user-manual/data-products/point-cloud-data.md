# Point Cloud Data

## Summary

3D point cloud data is collected using the Fraunhofer 3D laserscanner. .

## Data access

Data is available via Clowder and Globus.

* **Clowder**: [scanner3DTop collection](https://terraref.ncsa.illinois.edu/clowder/collection/5728c0a4e4b03269d7079ac0)
* **Globus path**:  `/sites/ua_mac/raw_data/scanner3DTop`
* **Sensor information**: [Fraunhofer 3D scanner collection](https://terraref.ncsa.illinois.edu/clowder/files/581793394f0ce77b66562ff9?dataset=581789af4f0ce77b6655d094&space=)

For details about using this data via Clowder or Globus, please see [Data Access](../how-to-access-data/) section.

## Computational pipeline

Raw sensor output \(PLY\) is converted to LAS format using the `ply2las` extractor

[**ply2las**](https://github.com/terraref/extractors-3dscanner)[ **extractor**](https://github.com/terraref/extractors-3dscanner)

* **Description**: PLY data is converted to LAS using the 3D point cloud extractor
* **Output**: 
  * **Clowder:** LAS file is added to the dataset
  * **Globus**: `/sites/ua_mac/Level_1/scanner3DTop`

## See also

* [Geospatial information](geospatial-information.md)

