# IPTK
This repositry contains the preliminary specification for the Imaging Pipeline Toolkit (IPTK). A reference implementation in Python is available at [iptk/iptk-py](https://github.com/iptk/iptk-py). This document is a work in progress and issues and pull requests against the spec are welcome.

## Foundations
At its core, IPTK is just a structured way of saving arbitrary data in a folder structure on disk. While we provide additional tools for web-based access, database indices, the truth is always stored within this simple structure, all other applications are required to eventually sync with the file system.

IPTK defines three core entities, _identifiers_, _datasets_, and _metadata_. Each dataset and each _type_ of metadata (called a _metadata specification_) have a unique identifier. Each dataset can be associated with no more than one set of metadata of each type.

## Identifiers
A valid identifier is a lowercase hexadecimal string of exactly 40 character, i.e. a string matching the regular expression `[0-9a-z]{40}`. There are no strict rules on how to generate identifiers for new datasets. Taking the SHA1 hash of some random data works just fine. Throughout this document we will use _\<id\>_ as a shorthand for a valid identifier.

## Datasets
An IPTK dataset is a folder containing arbitrary data stored in a defined format. The name of the dataset folder itself must be its identifier and it must contain the two subfolders _data/_ and _meta/_. It may additionally contain a folder called _lock/_. 

### The data/ subdirectory
The _data/_ folder contains the raw data of the dataset. It may contain arbitrary subfolders and files. E.g. for a DICOM dataset this may consist of all images of one study instance.

### The meta/ subdirectory
The _meta/_ folder may only contain metadata as specified below. It may not contain any subfolders and all files within must be named _\<id\>.json_, where the identifier is that of a valid metadata specification.

### The lock/ subdirectory
The _lock/_ directory indicates the state of the dataset. If it is present, IPTK-compatible programs will assume the contents of the dataset to be immutable. You may not edit or amend the contents of the _data/_ folder if _lock/_ exists.

It is not allowed to delete the _lock/_ folder, i.e. a locked dataset cannot be unlocked again. To edit a locked dataset, create a new empty dataset, copy the contents over, then edit the new dataset.

The metadata of locked datasets may still be edited without any restrictions.

### Sample
See below for an example of a valid IPTK dataset containing some DICOM images and two sets of associated metadata. The 

```
92024b2371150d11001491646e2c18390e702255
├── data
│   ├── slice_001.dcm
│   ├── slice_002.dcm
│   └── slice_003.dcm
├── meta
│   ├── 21926de7e59ea3818fa340090a295191b88c6b2b.json
│   └── 64e246bc37f9aa94ac13b848149ed750a4a689c6.json
└─── lock
```

## Metadata
Metadata within IPTK consists of JSON-serialized key-value-pairs stored within the _meta/_ subfolder of each dataset. Each metadata set is uniquely defined by its specification and its dataset.

### Metadata specifications
A metadata specification reserves an identifier for a specific kind of metadata. While you can simply generate a random identifier and use that for all your metadata of a specific kind, all users are encouraged to share their specifications at [iptk/metadata-specs](https://github.com/iptk/metadata-specs).

At a minimum, a metadata specification should contain the reserved _identifier_, a short _name_ and some _description_ of the kind of data that is stored under this specification. Additionally, a _contact_, _url_, and _organization_ may be specified, where further information about the metadata can be obtained. The name and description may be used in IPTK-based user interfaces.

Different, otherwise identical, versions of the same metadata specification can exist, if their identifiers are different.

#### Sample
```javascript
// Simple specification to reserve an identifier
{
	"name": "DICOM Tags",
	"description": "Values extracted from frequently used DICOM header fields.",
	"identifier": "52c1bba9c08888c2e530166b8bd1d62db76f89cc",
	"contact": "Jan-Gerd Tenberge <jan-gerd.tenberge@uni-muenster.de>",
	"organization": "University of Münster",
	"url": "https://example.com"
}

// Minimal specification
{
	"name": "Tags",
	"description": "User-defined tags are stored here.",
	"identifier": "2bc88bb1cbe97e9fa747ea54635888983de942d6"
}
```

### Metadata format
The following rules apply to metadata objects:

1. Each key must be a string.
2. Each value may only be a string, a boolean, a number, an array thereof, or _null_.
3. If the value is an array, all items must be of the same type.
4. A value cannot contain nested arrays or objects.

Date and time values should be saved as strings in the ISO 8601 format.


#### Samples
```javascript
// A valid set of metadata
{
	"patientName": "John Doe",
	"patientAge": 25,
	"patientWeight": 70.23,
	"parentNames": ["Jane Doe", "James Doe"],
	"dateOfBirth": "1992-10-04"
}

// Valid, but strongly discouraged (ambiguous date format)
{
	"dateOfBirth": "5/6/92"
}

// Invalid (value is an object)
{
	"patientDetails": {
		"age": 25,
		"name": "John Doe"
	}
}

// Invalid (nested array)
{
	"freeIntervals": [
		[1999, 2001], 
		[2004, 2017]
	]
}

```
