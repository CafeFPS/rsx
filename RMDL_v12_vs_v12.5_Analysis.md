# RMDL v12 vs v12.5 Format Analysis

## Overview

This document provides a comprehensive analysis of the differences between RMDL (R5 Model) version 12 and version 12.5 formats as implemented in the RSX codebase. The analysis is based on the source code found in `src/game/rtech/utils/studio/studio_r5_v12.h` and related files.

## Key Finding

**RMDL v12.5 is a minor incremental update from v12.4, adding only one additional field to the header structure.**

## Version Evolution Timeline

The v12 series had several incremental updates:

1. **v12.1** - Base v12 structure (`studiohdr_v12_1_t`)
2. **v12.2** - Added `int unk_v12_2` field (`studiohdr_v12_2_t`) 
3. **v12.3** - Added `int unk_20C[2]` array (`studiohdr_v12_4_t`)
4. **v12.4** - No new fields (same structure as v12.3)
5. **v12.5** - Added `int unk_214` field (`studiohdr_v12_5_t`)

## Structural Differences

### RMDL v12.4 (`studiohdr_v12_4_t`)
- **Size**: Ends with `int unk_20C[2]` (8 bytes)
- **Last fields**:
  ```cpp
  int unk_20C[2]; // added in rmdl v12.3
  ```

### RMDL v12.5 (`studiohdr_v12_5_t`)
- **Size**: Adds one additional field after v12.4 structure  
- **Additional field**:
  ```cpp
  int unk_20C[2]; // added in rmdl v12.3
  int unk_214;    // added in rmdl v12.4 (actually v12.5)
  ```

## Header Structure Comparison

Both versions share the exact same header structure up until the final field:

### Common Fields (both v12.4 and v12.5)
```cpp
struct studiohdr_v12_x_t
{
    int id;                    // Model format ID "IDST"
    int version;               // Format version number  
    int checksum;              // Checksum for validation
    int sznameindex;           // Name string offset
    char name[64];             // Internal model name
    int length;                // Data size of MDL file
    
    Vector eyeposition;        // Ideal eye position
    Vector illumposition;      // Illumination center
    Vector hull_min;           // Movement hull min
    Vector hull_max;           // Movement hull max
    Vector view_bbmin;         // Clipping bounding box min
    Vector view_bbmax;         // Clipping bounding box max
    
    int flags;
    
    // Bone data
    int numbones;
    int boneindex;
    int numbonecontrollers;
    int bonecontrollerindex;
    
    // Hit boxes
    int numhitboxsets;
    int hitboxsetindex;
    
    // Animations
    int numlocalanim;
    int localanimindex;
    int numlocalseq;
    int localseqindex;
    
    // ... (many more fields)
    
    // Version-specific additions:
    int unk_v12_2;             // Added in v12.2
    // ... (more fields)
    int unk_20C[2];            // Added in v12.3
    
    // v12.5 ONLY:
    int unk_214;               // Added in v12.5
};
```

## Processing Differences

### Code Handling
From `src/game/rtech/assets/model.cpp`, both v12.4 and v12.5 are handled identically:

```cpp
case eMDLVersion::VERSION_12_3:
case eMDLVersion::VERSION_12_4:
case eMDLVersion::VERSION_12_5:
{
    ModelAssetHeader_v12_1_t* hdr = reinterpret_cast<ModelAssetHeader_v12_1_t*>(pakAsset->header());
    mdlAsset = new ModelAsset(hdr, streamEntry, ver);
    
    ParseModelBoneData_v12_1(mdlAsset->GetParsedData());
    ParseModelAttachmentData_v8(mdlAsset->GetParsedData());
    ParseModelTextureData_v8(mdlAsset->GetParsedData());
    ParseModelVertexData_v12_1(pakAsset, mdlAsset);
    ParseModelSequenceData_Stall<r5::mstudioseqdesc_v8_t>(mdlAsset->GetParsedData(), reinterpret_cast<char* const>(mdlAsset->data));
    break;
}
```

### Version Detection
Version detection is based on header size comparison:

```cpp
// From model.h
if (pHdr[102] == sizeof(r5::studiohdr_v12_5_t) || pHdr[41] == sizeof(r5::studiohdr_v12_5_t))
    return eMDLVersion::VERSION_12_5;
```

## Data Section Differences

### Vertex Data
- **No differences** - Both versions use the same vertex data parsing (`ParseModelVertexData_v12_1`)

### Bone Data  
- **No differences** - Both versions use the same bone data parsing (`ParseModelBoneData_v12_1`)

### Animation Data
- **No differences** - Both versions use the same sequence parsing

### Texture Data
- **No differences** - Both versions use the same texture parsing (`ParseModelTextureData_v8`)

## Unknown Field Analysis

### `int unk_214` (v12.5 only)
- **Location**: Offset 0x214 in header
- **Size**: 4 bytes (32-bit integer)
- **Purpose**: Unknown - no usage found in codebase
- **Position**: Very last field in the header structure

This field appears to have been added for future functionality or to store additional metadata that wasn't present in v12.4.

## Backward Compatibility

RMDL v12.5 is **fully backward compatible** with v12.4 processing:
- Same parsing pipeline
- Same data structures used for processing
- Only difference is the additional 4-byte field at the end
- Applications reading v12.4 can read v12.5 by ignoring the last 4 bytes

## Summary

**RMDL v12.5 is a minimal format update that adds only one unknown integer field (`unk_214`) to the end of the v12.4 header structure.** 

The change is:
- **Structural**: +4 bytes to header size
- **Functional**: No changes to processing logic
- **Compatibility**: Fully backward compatible

The purpose of the additional field is currently unknown and may have been added for future functionality or engine-specific metadata storage.

## File Locations

- Header definitions: `src/game/rtech/utils/studio/studio_r5_v12.h`
- Version detection: `src/game/rtech/assets/model.h`  
- Processing logic: `src/game/rtech/assets/model.cpp`
- Generic handling: `src/game/rtech/utils/studio/studio_generic.cpp`

---

*Analysis completed based on RSX codebase examination*