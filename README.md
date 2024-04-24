# Multilevel Percolation (MLP) analysis

This page has been setup to support the preprint titled **_Geometry and organization of coherent structures in stably stratified atmospheric boundary layers_** which can be found in arXiv: https://arxiv.org/abs/2110.02253

This is an extension of of the percolation analysis approach described in Moisy & Jiménez 2004, JFM [1] and associated code available at https://github.com/Phoenixfire1081/PercolationAnalysis/. In the stably stratified atmospheric boundary layers, global intermittency forces turbulence to organize into patches. In such flow conditions, a single threshold obtained through percolation analysis is insufficient to identify individual coherent structures. Therefore, Harikrishnan et al., 2021 [2] proposed a novel procedure to make percolation analysis iterative, thus breaking down complex structures over and over until simple structures remain. For definitions on simple, complex structures and the procedure of MLP itself, please refer to the work of Harikrishnan et al., 2021 [2].

If you use this code for your work, please cite our arXiv preprint [4]:
```
@article{harikrishnan2021geometry,
  title={Geometry and organization of coherent structures in stably stratified atmospheric boundary layers},
  author={Harikrishnan, Abhishek and Ansorge, Cedrick and Klein, Rupert and Vercauteren, Nikki},
  journal={arXiv preprint arXiv:2110.02253},
  year={2021}
}
```

## Installation

The code is available as a package from PyPI: https://pypi.org/project/multiLevelPercolation/. It is dependent on two other packages: boxcounting, percolationAnalysis.

```
pip install boxcounting percolationAnalysis multiLevelPercolation
```

## Example - Calculating unique thresholds for structures from the given test data

Download testData.bin from the GitHub website first.

```
import array
import numpy as np
from multiLevelPercolation import multiLevelPercolation
from extractStructuresWithMC import extractStructuresMC

# Select file to run tests on
_filenameRead = 'testData.bin'

# What indexing does it use?
# This refers to array order
# See here for a full description: https://docs.oracle.com/cd/E19957-01/805-4940/z400091044d0/index.html
# Python uses C-order indexing (or row-major order). For this set _zFastest = True
_zFastest = False

# Set starting threshold
# This is obtained through percolation analysis
_threshVal = 0.32

# Set additional parameters

# Writes information relating to percolation analysis
_writePercolationData = False

# Writes structure location information.
# Useful for visualization purposes
_writeNeighborInformation = False

# Set data related parameters
xlen = 100 
ylen = 50
zlen = 200

# Set precision of binary data. 'f' is 32-bit floating point data.
# For others, see here: https://docs.python.org/3/library/array.html
precision = 'f'

# Use array to read data. This is fast for binary files.
data = array.array(precision)
fr = open(_filenameRead, 'rb')
data.fromfile(fr, (xlen*ylen*zlen))
fr.close()

# Convert data to _zFastest before processing
if not _zFastest:
	data = np.reshape(data, [xlen, ylen, zlen], order = 'F')
	data = data.ravel()
	_zFastest = True
	# NOTE: This conversion means that Simple structures will be in zFastest
	# If simple structures are necessary in xFastest, read the data and do a transpose

# Print out details
_verbose = True

# Use Marching Cubes neighbor correction (more computation power required)
_marchingCubesExt = True

# Set datatype for new grids
_dataType = np.float32

# Filter structures less than a specific fractal dimension
# This ensures noise-free result
lessFD = True

# Set filter value
lessFDval = 1

# Set minimum ratio of Vmax/V
# Structures having minimum value of the ratio Vmax/V, where
# Vmax - Volume of the biggest structure in the domain
# V - Volume of all structures in the domain,
# greater than the set value will be classified as simple
# Others will be classified complex
# Generally 0.5 admits at most two structures as simple, one of
# which has a larger volume than the other 
minVmVval = 0.5

# Clean up percolation data
_cleanup = True

fw = open('log.txt', 'w+')
fw.write(_filenameRead + '\n')
fw.close()

MLPObj = multiLevelPercolation(_threshVal, data, xlen, ylen, zlen, _zFastest, \
_verbose, _marchingCubesExt, _dataType)
MLPObj.getSimpleStructures(lessFD, lessFDval, minVmVval, _cleanup)

# Get individual thresholds
data = array.array(precision)
fr = open('SimpleStructures.bin', 'rb')
data.fromfile(fr, (xlen*ylen*zlen))
fr.close()

# Both data are not necessary
_writeNeighborInformation = False
_writePercolationData = False

# Data is in zFastest always
data = np.reshape(data, [xlen, ylen, zlen])
extractStructuresObj = extractStructuresMC(_threshVal, data, \
xlen, ylen, zlen, _zFastest, _verbose, \
_writeNeighborInformation, _writePercolationData, _marchingCubesExt)
structureGrid = extractStructuresObj.extract()

# Get individual thresholds
allThresholds = MLPObj.getIndividualThresholds(data, structureGrid)

fw = open('IndividualThresholds.txt', 'w+')
for i in range(len(allThresholds)):
	fw.write(str(i+1) + ' ' + str(allThresholds[i]) + '\n')
fw.close()
```

## References

[1] Moisy, Frédéric, and Javier Jiménez. "Geometry and clustering of intense structures in isotropic turbulence." Journal of fluid mechanics 513 (2004): 111-133.

[2] Harikrishnan, Abhishek, et al. "Geometry and organization of coherent structures in stably stratified atmospheric boundary layers." arXiv preprint arXiv:2110.02253 (2021).
