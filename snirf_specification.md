
Shared Near Infrared File Format Specification
============================================================

### License: This document is in the public domain.

The file format specification uses the extension \*.snirf.  These are HDF5 
format files, renamed with the .snirf extension.  For a program to be 
“SNIRF-compliant”, it must be able to read and write the SNIRF file.  

The structure of each data file has a minimum of 5 required elements. For 
each element in the data structure, one of the 4 types is assigned, including
- `container`: a structure containing sub-fields
- `string`: a UTF-8 (ISO 10646) encoded string
- `integer`: one of the integer types (1-byte, 2-byte, 4-byte, 8-byte)
- `numeric`: one of the floating-point types (8-byte, 4-byte, 2-byte)

For `integer` and `numeric` data fields, users should use HDF5's 
Datatype Interface to query the byte-length stored in the file.

## SNIRF file specification
### Required fields 
<dl>
<dt>formatVersion</dt><tt>[Type: string]</tt>
<dd>This is a string that specifies the version of the file format.  This 
document describes format version “1.0”</dd>

<dt>data(n)</dt><tt>[Type: container]</tt>
<dd>This is a structure containing the data <i>dataTimeSeries</i>, the time vector <i>time</i> of when the 
samples were acquired, and a description of the channels used to acquire the 
data. The data can be grouped in blocks indexed by index <i>n</i>. One convenient approach 
to blocking the data is to group all channels with the same time vector <i>time</i>.</dd>

<dt>data(n).dataTimeSeries</dt><tt>[Type: numeric]</tt>
<dd>This is the actual raw data variable. This variable has dimensions of 
<tt>&lt;number of time points&gt; x &lt;number of channels&gt;</tt>.   
Columns in <i>dataTimeSeries</i> are mapped to the measurement list (<i>measurementList</i> variable described below).  
The <i>dataTimeSeries</i> variable can be complex (as in the case of sine-cosine demodulation for 
the laser carrier frequencies).</dd>

<dt>data(n).time</dt><tt>[Type: numeric]</tt>
<dd>The <i>time</i> variable. This provides the acquisition time of the measurement 
relative to the time origin.  This will usually be a straight line with slope 
equal to the acquisition frequency, but does not need to be equal spacing.  The 
size of this variable is <tt>&lt;number of time points&gt; x 1</tt>.</dd>

<dt>data(n).measurementList</dt><tt>[Type: container]</tt>
<dd>The measurement list. This variable serves to map the data array onto the 
probe geometry (sources and detectors), data type, and wavelength.   This 
variable is an array structure that has the size <tt>&lt;number of 
channels&gt;</tt> that describes the corresponding column in the data matrix. 

For example, the *measurementList(3)* describes the third column of the data matrix (i.e. 
*dataTimeSeries(:,3)*).
	
Each element of the array is a structure which describes the measurement 
conditions for this data with the following fields:
* *measurementList(k).sourceIndex*<tt>[Type: integer]</tt>: index (starting from 1) of the source
* *measurementList(k).detectorIndex*<tt>[Type: integer]</tt>: index (starting from 1) of the detector
* *measurementList(k).wavelengthIndex*<tt>[Type: integer]</tt>: index (starting from 1) of the wavelength
* *measurementList(k).dataType*<tt>[Type: integer]</tt>: data-type identifier, see Appendix
* *measurementList(k).dataTypeIndex*<tt>[Type: integer]</tt>: data-type specific parameter indices

Optional fields include:
* *measurementList(k).sourcePower*<tt>[Type: numeric]</tt>: source power in milliwatt (mW)
* *measurementList(k).detectorGain*<tt>[Type: numeric]</tt>: detector gain
* *measurementList(k).moduleIndex*<tt>[Type: integer]</tt>: index (starting from 1) of a repeating module

For example, if *measurementList(5)* is a structure with *sourceIndex=2*, *detectorIndex=3*, 
*wavelengthIndex=1*, *dataType=1*, *dataTypeIndex=1* would imply that the data 
in the 5th column of the *dataTimeSeries* variable was measured with source #2 and detector 
#3 at wavelength #1.  Wavelengths (in nanometers) are described in the 
*probe.wavelengths* variable  (described later). The data type in this case is 1, 
implying that it was a continuous wave measurement.  The complete list of 
currently supported data types is found in the Appendix. The data type index 
specifies additional data type specific parameters that are further elaborated 
by other fields in the *probe* structure, as detailed below. Note that the Time 
Domain and Diffuse Correlation Spectroscopy data types have two additional 
parameters and so the data type index must be a vector with 2 elements that 
index the additional parameters.

sourcePower provides the option for information about the source power for that 
channel to be saved along with the data. The units are not defined, unless the 
user takes the option of using a *metaDataTag* described below to define, for 
instance, *sourcePowerUnit*. *detectorGain* provides the option for information 
about the detector gain for that channel to be saved along with the data.

Note:  The source indices generally refer to the optode naming (probe 
positions) and not necessarily the physical laser numbers on the instrument. 
The same is true for the detector indices.  Each source optode would generally, 
but not necessarily, have 2 or more wavelengths (hence lasers) plugged into it 
in order to calculate deoxy- and oxy-hemoglobin concentrations. The data from 
these two wavelengths will be indexed by the same source, detector, and data 
type values, but have different wavelength indices. Using the same source index 
for lasers at the same location but with different wavelengths simplifies the 
bookkeeping for converting intensity measurements into concentration changes. 
As described below, optional variables *probe.sourceLabels* and *probe.detectorLabels* are 
provided for indicating the instrument specific label for sources and detectors.
</dd>

<dt>stim</dt><tt>[Type: container]</tt>
<dd>This is an array describing any stimulus conditions. Each element of the 
array has the following required fields.</dd>

<dl>
<dt>stim(n).name</dt><tt>[Type: string]</tt>
<dd>This is a string describing the n<sup>th</sup> stimulus condition.</dd>

<dt>stim(n).data</dt><tt>[Type: numeric]</tt>
<dd> This is a three-column array specifying the stimulus time course for the 
n<sup>th</sup> condition. Each row corresponds with a specific stimulus trial. 
The three columns indicate [starttime duration value].  The starttime, in 
seconds, is the time relative to the time origin when the stimulus takes on a 
value; the duration is the time in seconds that the stimulus value continues, 
and value is the stimulus amplitude.  The number of rows is not constrained. 
(see examples in the appendix).</dd>
</dl>

<dt>probe</dt><tt>[Type: container]</tt>
<dd>This is a structured variable that describes the probe (source-detector) 
geometry.  This variable has a number of required fields.</dd>

<dl>
<dt>probe.wavelengths</dt><tt>[Type: numeric]</tt>
<dd>This field describes the wavelengths used.  This is indexed by the 
wavelength index of the measurementList variable.
	
For example, *probe.wavelengths* = [690 780 830]; implies that the measurements were 
taken at three wavelengths (690nm, 780nm, and 830nm).  The wavelength index of 
*measurementList(k).wavelengthIndex* variable refers to this field.  *measurementList(k).wavelengthIndex* 
= 2 means the k<sup>th</sup> measurement was at 780nm.

The number of wavelengths is not limited (except that at least two are needed 
to calculate the two forms of hemoglobin).  Each source-detector pair would 
generally have measurements at all wavelengths.
</dd>
<dt>probe.wavelengthsEmission</dt><tt>[Type: numeric]</tt>
<dd>This field is required only for fluorescence data types, and describes the 
emission wavelengths used.  The indexing of this variable is the same 
wavelength index in measurementList used for <i>probe.wavelengths</i> such that the excitation wavelength 
is paired with this emission wavelength for a given measurement.</dd>

<dt>probe.sourcePos</dt><tt>[Type: numeric]</tt>
<dd>This field describes the position (in spatialUnit units) of each source 
optode.  This field has size <tt>&lt;number of sources&gt; x 3</tt>.  For 
example, <i>probe.sourcePos(1,:)</i> = [1.4 1 0], and <i>SpatialUnit='cm'</i>; places source 
number 1 at x=1.4 cm and y=1 cm and z=0 cm.

Dimensions are relative coordinates (i.e. to some arbitrary defined origin).  
The *qform* variable described below can be used to define the transformation 
between this SNIRF coordinate system and other coordinate systems.
</dd>

<dt>probe.detectorPos</dt><tt>[Type: numeric]</tt>
<dd>Same as <i>probe.sourcePos</i>, but describing the detector positions.</dd>
</dl>
There are additional required elements of the <i>probe</i> structure, depending on the 
data type of the measurement.  These variables are indexed by 
<i>measurementList(k).dataTypeIndex</i>:

<dl>
Continuous wave (Fluorescence or non-fluorescence):

- *None*

Frequency Domain (Fluorescence or non-fluorescence):
- *probe.frequency*<tt>[Type: numeric]</tt>: modulation frequency in Hz

Time domain – gated (Fluorescence or non-fluorescence):
- *probe.timeDelay*<tt>[Type: numeric]</tt>
- *probe.timeDelayWidth*<tt>[Type: numeric]</tt>

Time domain – moments (Fluorescence or non-fluorescence):
- *probe.momentOrder*<tt>[Type: numeric]</tt>

Diffuse Correlation spectroscopy (Fluorescence or non-fluorescence):
- *probe.correlationTimeDelay*<tt>[Type: numeric]</tt>
- *probe.correlationTimeDelayWidth*<tt>[Type: numeric]</tt>
</dl>

There are optional fields of the *probe* structure that can be used.

<dl>
<dt>probe.sourceLabels</dt><tt>[Type: string]</tt>
<dd>This is a string array providing user friendly or instrument specific 
labels for each source. Each element of the array must be a unique string
among both <i>probe.sourceLabels</i> and <i>probe.detectorLabels</i>.
This can be of size <tt>&lt;number of sources&gt;
x 1</tt> or <tt>&lt;number of sources&gt; x &lt;number of 
wavelengths&gt;</tt>. This is indexed by <i>measurementList(k).sourceIndex</i> and 
<i>measurementList(k).wavelengthIndex</i>.</dd>

<dt>probe.detectorLabels</dt><tt>[Type: string]</tt>
<dd>This is a string array providing user friendly or instrument specific 
labels for each detector. Each element of the array must be a unique string
among both <i>probe.sourceLabels</i> and <i>probe.detectorLabels</i>.
This is indexed by <i>measurementList(k).detectorIndex</i>.</dd>

<dt>probe.landmark</dt><tt>[Type: numeric]</tt>
<dd>This is a 2-D array storing the neurological landmark positions measurement
from 3-D digitization and tracking systems to facilitate the registration and 
mapping of optical data to brain anatomy. This array should contain a minimum 
of 3 columns, representing the x, y and z coordinates of the digitized landmark
positions. If a 4th column presents, it stores the index to the labels of the 
given landmark. Label names are stored in the <i>probe.landmarkLabels</i> subfield.
An label index of 0 refers to an undefined landmark. </dd>

<dt>probe.landmarkLabels</dt><tt>[Type: container]</tt>
<dd>This string array stores the names of the landmarks. The first string 
denotes the name of the landmarks with an index of 1 in the 4th column of 
<i>probe.landmark</i>, and so on. One can adopt the commonly used 10-20 landmark 
names, such as "Nasion", "Inion", "Cz" etc, or use user-defined landmark 
labels. The landmark label can also use the unique source and detector labels
defined in <i>probe.sourceLabels</i> and <i>probe.detectorLabels</i>, respectively, to 
associate the given landmark to a specific source or detector. All 
strings are UTF-8 encoded.</dd>
<dl>

<dt>probe.useLocalIndex</dt><tt>[Type: integer]</tt>
<dd>For modular fNIRS systems, setting this flag to a non-zero integer indicates
that <i>measurementList(k).sourceIndex</i> and <i>measurementList(k).detectorIndex</i> are module-specific
local-indices. One must also include <i>measurementList(k).moduleIndex</i> in the <i>measurementList</i>
structure in order to restore the global indices of the sources/detectors.</dd>

<dt>metaDataTags</dt><tt>[Type: container]</tt>
<dd>This is a two column string array of arbitrary length consisting of any 
key/value pairs the user (or manufacturer) would like to put in.  Each row of 
the array consists of two strings. Some possible examples:

```
['ManufacturerName','ISS'],
['Model','Imagent'],
['SubjectName', 'Pseudonym, I.M.A.'],
['DateOfBirth','20120401'],
['AcquisitionStartTime','150127.34']
['StudyID','Infant Brain Development']
['StudyDescription','We study infant cognitive development.']
['AccessionNumber','INA2S12']
['InstanceNumber','2']
['CalibrationFileName','phantomcal_121015.snirf']
```
While these tags are freeform, some conventions must be followed.  Keys should 
use only alphanumeric characters with no spaces, with individual words 
capitalized.  All values will be stored as strings, How strings are converted 
into numeric values is left to whoever defines the Key.  However, it is 
required that dates be stored as YYYYMMDD, and clock times be stored as 
HHMMSS.SSSS… (24 hour format) for consistency.  Time intervals must be in 
seconds.

The following metadata tags are required:
```
SubjectID
MeasurementDate
MeasurementTime
SpatialUnit    (allowed values are 'mm' and 'cm')
```
</dd>

The metadata tags "StudyID" and "AccessionNumber" are unique strings that can 
be used to link the current dataset to a particular study and a particular
procedure, respectively. The "StudyID" tag is similar to the DICOM tag 
"Study ID" (0020,0010) and "AccessionNumber" is similar to the DICOM tag 
"Accession Number"(0008,0050), as defined in the DICOM standard (ISO 12052).

The metadata tag "InstanceNumber" is defined similarly to the DICOM tag 
"Instance Number" (0020,0013), and can be used as the sequence number to
group multiple datasets into a larger dataset - for example, concatenating 
streamed data segments during a long measurement session.


### Optional variables:
These variables are not required for basic functions, but might be useful to 
get more out of your data sets.

<dl>
<dt>aux</dt><tt>[Type: container]</tt>
<dd>This optional array specifies any recorded auxiliary data. Each element of 
<i>aux</i> has the following required fields:</dd>

<dt>aux(n).name</dt><tt>[Type: string]</tt>
<dd>This is string describing the n<sup>th</sup> auxiliary data timecourse.</dd>

<dt>aux(n).dataTimeSeries</dt><tt>[Type: numeric]</tt>
<dd>This is the aux data variable. This variable has dimensions of 
<tt>&lt;number of time points&gt; x 1</tt>.</dd>

<dt>aux(n).time</dt><tt>[Type: numeric]</tt>
<dd>The time variable. This provides the acquisition time of the aux 
measurement relative to the time origin.  This will usually be a straight line 
with slope equal to the acquisition frequency, but does not need to be equal 
spacing.  The size of this variable is <tt>&lt;number of time points&gt; x 1</tt>.</dd>

<dt>timeOffset</dt><tt>[Type: numeric]</tt>
<dd>This variable specifies the offset of the file time origin relative to 
absolute (clock) time in seconds.</dd>


## Appendix

Supported data types for “dataTimeSeries”
1.	Raw - Continuous wave
2.	Raw - Frequency Domain
3.	Raw - Time domain - gated
4.	Raw - Time domain – moments
5.	Raw - Diffuse Correlation spectroscopy
6.	Raw - Fluorescence – continuous wave 
7.	Raw - Fluorescence - Frequency Domain
8.	Raw - Fluorescence - Time domain - gated
9.	Raw - Fluorescence - Time domain – moments
10.	Raw - Bioluminescence – continuous wave

Examples of stimulus waveforms
Assume there are 10 time points, starting at zero, spaced 0.1s apart.  If we 
assume a stimulus to be a 0.2 second off, 0.2 second on repeating block, it 
would be specified as follows:
```
    [0.2 0.2 1.0]
    [0.6 0.2 1.0]
```

## Acknowledgement

This document was originally drafted by Blaise Frederic (bbfrederick at mclean.harvard.edu) and David Boas (dboas at bu.edu).

Other significant contributors to this specification include:
- Theodore Huppert (huppert1 at pitt.edu)
- Jay Dubb (jdubb at bu.edu)
- Qianqian Fang (q.fang at neu.edu)

The following individuals representing academic, industrial, software, and hardware interests are also contributing to and supporting the adoption of this specification:

### Software
- Adam Eggebrecht, University of Washington, neuroDOT
- Felipe Orihuela-Espina, Instituto Nacional de Astrofísica, Óptica y Electrónica, ICNNA
- Sungho Tak, Korea Basic Science Institute, NIR-SPM
- Luca Pollonini, Houston Methodist, Phoebe
- Hamid Deghani, University of Birmingham, NIRFAST
- Stanislaw Wojtkiewicz, University of Birmingham, NIRFAST
- Joe Culver, University of Washington, neuroDOT
- Christophe Grova, McGill University, NIRS Storm
- Ata Akin, Acıbadem University
- Alessandro Torricelli, Politecnico di Milano

### Hardware
- Jorn Horschig, Artinis Inc
- Hasan Ayaz, Biopac Inc
- Rob Cooper, Gower Labs Inc
- Rueben Hill, Gower Labs Inc
- Hanseok Yun, OdeLab Inc
- Hirokazu Asaka, Hitachi
- Takumi Inakazu, Hitachi
- Lamija Pasalic, NIRx
- Mathieu Coursolle, Rogue
