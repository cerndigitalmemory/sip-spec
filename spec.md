# CERN SIP DRAFT 1

This specification aims to implement the 4.2.2.2/Submission Information Package (SIP) reference definition from the OAIS Reference Model (Pink pre-magenta book / September 2019 - ISO 14721).

- [GLOSSARY](#GLOSSARY)
- [SCOPE](#SCOPE)
- [CERN SIP Specification](#CERN-SIP-Specification)
- [EXAMPLES](#EXAMPLES)

### GLOSSARY

- Consumer - The role played by those persons, or client systems, who interact with OAIS services to find preserved information of interest and to access that information in whatever level of detail is allowed. 
- Producer - The role played by those persons or client systems that provide the information to be preserved.
- OAIS - The system.


### SCOPE

**Submission Information Packages (SIPs)** are the packages sent by Producers to the CERN OAIS. It is the purpose of this document to provide a formal specification of how these packages must be produced.

In practise, this represents the (initial) negotiation between the information Producer and the OAIS on the form and the details of the SIPs. Additional steps must be taken (such as SIGNING) and more requisites can be agreed upon.

A SIP consists in Content Information and various kind of information necessary to assure that those data can be maintained by the OAIS and that the data can be interpreted and consumed by Consumers who access the information from the OAIS in the future.

Each SIP must provide:

1. **Content Information**: the original target of preservation. It's composed of 
    a. the **Content Data Object** (the original bytestream) 
    b. and its **Representation Information**.


Content Information is further described by:

2. **PDI** Preservation Description Information (with its **Representation Information**)
    The PDI must include information that is necessary to adequately preserve the particular Content Data Object with which it is associated:
    a. **Reference Information** - identifies, and if necessary describes, one or more mechanisms used to provide assigned identifiers for the Content Data Object. It also provides those identifiers that allow outside systems to refer, unambiguously, to this particular Content Data Object.
    b. **Context Information** - documents the relationships of the Content Data Object to its environment.
    c. **Provenance Information** - documents the history of the Content Data Object. This tells the origin or source of the Content Data Object, its Information Properties to be preserved (Transformational Information Properties), any changes that may have taken place since it was originated, and who has had custody of it since it was originated, providing an audit trail for the Content Data Object.
    d. **Fixity Information** - provides the data integrity checks or validation/verification keys used to ensure that the particular Content Data Object has not been altered in an undocumented manner.
    e. **Access Right Information** - identifies the access restrictions pertaining to the Content Data Object, including the legal framework, licensing terms, and access control.


To help understand in which ways the following specification provides this information, each of these requirements are pointed at by "markers" enclosed in square brackets (e.g. the file implementing the point 1.a here is marked with **[1a]**). Some requisites can be satisfied by more than one element.

A SIP usually contains *some* of the PDI, while more can be added in the AIP (e.g. metadata about the digitation or preservation process, to be added as "Provenance Information" in the AIP).



### CERN SIP Specification

If not otherwise specified (e.g. marked with "optional"), each element described here is mandatory. The SIP won't be validated and accepted in the OAIS if those requirements aren't satified.


A CERN SIP consists in a folder conforming to the **BagIt File Packaging Format** specification ([V0.97](https://tools.ietf.org/id/draft-kunze-bagit-09.html)). Contents of such bags are provided and organised according to additional requirements and specification.

BagIt is an Internet Draft that specifies a file system structure for transferring and archiving a collection of files, including their checksums and brief metadata.

A BagIt bag can be seen as a mechanism for serialization and transport consistency, while the internal SIP structure specification (the contents of data folder) is a way to consistently provide identity, annotations, provenance, attached files and metadata of the resource target of the preservation process.

As such, the two formats complement each-other.



This "bag" is made of the following elements:

- A `data/` folder, containing 2 sub-folders:
    - `content`, containing the original archival target, in the original directory structure. [**1a**] E.g.:
    - `meta`, containing:
        -  `sip.json` [**2c, 2d, 2a, 1b**]
        -  the (upstream) metadata about the Data Object. This can consist of more than one file. Everything else in this folder a part from `sip.json` will be considered as part this category.
- `bagit.txt`, including the bagit version and the character encoding [**1b**]
    ```
    BagIt-Version: 0.97
    Tag-File-Character-Encoding: UTF-8
    ```
- `bag-info.txt` [**2c**]
    Some *basic* and additional metadata about the generation of the SIP package. These values (and more) are also specified in the "sip-process" field of the sip.json file. The only required value here is the "Payload Oxum".
    ```
    Bag-Software-Agent: toolname 0.1
    Bagging-Date: 2021-08-20
    Payload-Oxum: 5821296.63
    ```
- At least one manifest file that itemizes the filenames present in the “data” directory, as well as their checksums. [**2d**]
    ```
    data/content/42.mp4 HASH
    data/content/sachiel.png HASH
    ```

E.g.:
```
.
├── bag-info.txt
├── bagit.txt
├── data
│   ├── content
│   │   ├── 42.mp4
│   │   └── sachiel.png
│   └── meta
│       ├── metadata.xml
│       └── sip.json
└── manifest-md5.txt
```

### sip.json

The `sip.json` file must provide some high level metadata about the resource target of the archival process.

- `sourceid`. Unique identifier for the upstream sorce digital repository. This value must be negotiated beforehand.
- `recid`. Unique identifier for the resource (for the specified sourceid)
- `metadataFile_upstream`. Upstream URLs (e.g. API endpoints) where the metadata files were obtained.
- `contentfiles`. Details about files and exposes the relationship between the metadata files and the payload files.
- `timestamp` 
- `sip-process` information about the SIP generation process

A formal JSON schema specification for this file is provided [here]().

### VERSIONING

Versioning is enabled by submitting subsequents SIP(s) with the same `recid` and `sourceid` but a different `timestamp`.

E.g. 

### SERIALISATION

The SIP can be optionally serialized as a ZIP or TAR file. Several rules govern such serialization (as specified by the [BagIt specification Draft 14 section 4](https://datatracker.ietf.org/doc/html/draft-kunze-bagit-14#section-4) "Serialisation"):
   1.  The top-level directory of a serialization MUST contain only one
       bag.

   2.  The serialization SHOULD have the same name as the bag's base
       directory, but MUST have an extension added to identify the
       format.  For example, the receiver of "mybag.tar.gz" expects the
       corresponding base directory to be created as "mybag".

   3.  A bag MUST NOT be serialized from within its base directory, but
       from the parent of the base directory (where the base directory
       appears as an entry).  Thus, after a bag is deserialized in an
       empty directory, a listing of that directory shows exactly one
       entry.  For example, deserializing "mybag.zip" in an empty
       directory causes the creation of the base directory "mybag" and,
       beneath "mybag", the creation of all payload and tag files.

   4.  The deserialization of a bag MUST produce a single base directory
       bag with the top-level structure as described in this
       specification without requiring any additional un-archiving step. For example, after one un-archiving step it would be an error for
       the "data/" directory to appear as "data.tar.gz".  TAR and ZIP
       files may appear inside the payload beneath the "data/"
       directory, where they would be treated as any other payload file.
       
### TRUSTING

The Producer must negotiate a way to verify its identity. After having provided a public key the SIP (or part of it, e.g. `meta`) will be required to be signed to authenticate the Producer.

### OAIS Utils

The "OAIS Utils" python package provides some CLI tools to verify if a folder is a valid CERN SIP:

```
$ verify-sip my-generated-sip

This is not a valid CERN SIP according to specification DRAFT1: 
- sip.json is missing the timestamp key
```

### bagit-create

The bagit create python package provides a CLI tool to automatically generate CERN SIP for resources from Zenodo, the CERN Document Server, the CERN Open Data portal and similar digital repositories based on Invenio 1.x and 3.x.

```
$ bic --recid 2751237 --source cds 
```

### EXAMPLES
