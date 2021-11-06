# Plan for SONAR-netCDF4 convention adherence work in 0.6.0

Disruptive changes are preced by the prefixed **(D)**

- **(D)** `Beam` subgroup(s). echopype's move of `Beam` subgroup to a top-level group. Move it back into `Sonar`, as `Sonar/Beams` (or a slightly different name?)
- **(D)** Add a `beam` dimension to `Beam` group
  - For single beam or split-beam, it could be a length-1 explicit dimension or an implicit coordinate (scalar `beam` )
  - For multi-beam, an explicit dimension is required. Currently the EK80 parser generates such a structure, but the dimension is named `quadrant` (WJ: there is an extra dimension quadrant — you can call them separate beams, but since Simrad use quadrant i used that for our first shot though I think it should be probably sector since it is not always 4)
  - EM: It occurs to me now that, in terms of match-up with SONAR-netCDF4, the Beam group in AZFP and EK60 echopype nc/zarr files could be described as having an implicit, single-value `beam` dimension. Single-value dimensions can be implemented interchangeably in a netcdf file either explicitly with the dimension or implicitly (refer to the CF convention). In SONAR-netCDF4, since there is an explicit `beam` dimension, single-beam data would be expected to be stored with `beam` as a dimension.
  - issue of convention use of `beam` dimension (ie, >1 beams per Beam subgroup? Based on "beam mode" alone?)
  - In the convention, under 2.10.6 (Sonar groups), it says: "Variable definitions for data from split-aperture systems are not currently specified"
- `range_bin` issues
  - `range_bin` is not in the convention, but it's central to the echopype data structure
  - [Currently this coordinate variables doesn't have attributes.](https://github.com/OSOceanAcoustics/echopype/issues/373) Add units, long name, etc
  - small TODO, WJ's "convention vlen" comments. See also [this discussion in the SONAR-netCDF4 github repo](https://github.com/ices-publications/SONAR-netCDF4/issues/28) that directly addresses that aspect of the convention, and potential problems with it
- **(D)** Rename `time` dimensions and coordinates in `Platform` group, to adhere to the convention naming guideline. `location_time` does not comply; it should be `time1`, `time2`, etc
- [Compliance of variable encoding (`frequency` only?) in converted files, #307](https://github.com/OSOceanAcoustics/echopype/issues/307)
- Restart the element-by-element compliance check tables. See section below
- **Improved global attributes**
  - `survey_name` top-level attribute is not in the convention
  - When an attribute is empty (specially global attributes), should it still occur but with a blank value, or be ommitted altogether? Current implementation is inconsistent
  - Add missing core CF attributes (feature_type, cdm, etc), where appropriate
  - Add some useful attributes from ACDD, specially ones involving spatial and temporay bounding boxes; roles; etc; in top-level group? Will improve discoverability.
  - See AcMeta to see if we can define an initial subset of it that ideally follows the ACDD attributes we'd be interested in
- **Go over other relevant discussions and notes, to see if I'm missing anything**
  - Google Drive files (mappings, sample nc files, etc) under [echopype/convention_check](https://drive.google.com/drive/u/0/folders/1MPBRzrehXk9qAt8ZuBjFzR2Xh-0WFXXU)
  - [Document and verify compliance with SONAR-netCDF4 convention #210](https://github.com/OSOceanAcoustics/echopype/issues/210). Though we've closed this issue b/c it became sprawling, it still contains many important details that we'll refer back to.
  - echopype open issues with a [conventions](https://github.com/OSOceanAcoustics/echopype/issues?q=is%3Aissue+is%3Aopen+label%3Aconventions) flag.
  - echopype documentation: [Data format](https://echopype.readthedocs.io/en/stable/data-format.html) and [Why echopype?](https://echopype.readthedocs.io/en/stable/why.html)
  - `conventions/SONAR-netCDF4_compliance.ipynb`: code and file instrospection, and comments I've added there
  - I think I've already captured notes from these sources:
    - `sonar-conventions` Slack channel in `uw-echospace` workspace. Going back to the start of the channel in April '21
    - My hand written notes on the SONAR-netCDF4 report

## Convention compliance assessment and corrections by sensor

- From an earlier plan: We've laid out the sequence of tasks as follows:
  1. 1:1 mappings of variables, attributes and groups. Includes things only found in one of them (?).
  2. Content of variables, attributes and groups
  3. Things that are intentionally different, or diverged unintentionally
- Google Drive files (mappings, sample nc files, etc) under [echopype/convention_check](https://drive.google.com/drive/u/0/folders/1MPBRzrehXk9qAt8ZuBjFzR2Xh-0WFXXU)
- See the preliminary, per-sensor compliance Google sheets; eg, [EK60](https://docs.google.com/spreadsheets/d/1DXQtYPr-k7BDwaAa2a-yDO8sxt7QUSU6P-icedGKkX0/edit#gid=0)
- **Fix current errors/issues with EK60**
  - `Sonar/sonar_serial_number` is empty. See [issue 212](https://github.com/OSOceanAcoustics/echopype/issues/212)
  - Populate `Sonar/sonar_software_name`. Right now it's left blank; see [convert/set_groups_ek60.py#L89](https://github.com/OSOceanAcoustics/echopype/blob/class-redesign/echopype/convert/set_groups_ek60.py#L89) and [issue 210 comment](https://github.com/OSOceanAcoustics/echopype/issues/210#issuecomment-822642399)
  - `Sonar/sonar_model attribute` entry is "ER60". It should be EK60. Currently EK60 `Sonar/sonar_model` is set in [convert/set_groups_ek60.py#L87](https://github.com/OSOceanAcoustics/echopype/blob/class-redesign/echopype/convert/set_groups_ek60.py#L87) as `self.parser_obj.config_datagram['sounder_name']`, so that looks fine, unless the datagram `sounder_name` is parsed incorrectly. See [issue 210 comment](https://github.com/OSOceanAcoustics/echopype/issues/210#issuecomment-821761942)
  - *(NOTE: This issue may be broader than EK60)* `Provenance > src_filenames` should be a list of strings per the convention, but currently it's a string (actually, it's a string only when there's only one filename; it's turned into a list of strings after `combine_echodata`). It's also mispelled, as it should be `source_filenames`.
- **Reassess AZFP compliance, and primary issues**
  - There are many first-order issues!
  - Some of the comments under EK60 also apply to AZFP
- **Assess EK80**
  - I haven't assessed it at all. Use one of Wu-Jung's new, small EK80 raw files to make this assessment

## SONAR-netCDF4 References

- https://github.com/ices-publications/SONAR-netCDF4/ (see the "latest working draft" linked on the README file)
- [Published SONAR-netCDF4 version 1.0 report](http://www.ices.dk/sites/pub/Publication%20Reports/Cooperative%20Research%20Report%20(CRR)/CRR341.pdf)
- [Preview of the latest working draft ("version 2.0")](https://gitcdn.link/repo/ices-publications/SONAR-netCDF4/master/Formatted_docs/crr341.html)
- My spreadsheet version of the SONAR-netCDF4-CDLtables CDL-like tables from the vers. 1.0 report: [SONAR-netCDF4-CDLtables.xlsx](https://docs.google.com/spreadsheets/d/13iitnfHVdX_4tTNjUP99YKwgNrrBYEjv/edit#gid=1702761048)
- [convention_definition.ncml](https://github.com/ices-publications/SONAR-netCDF4/blob/master/docs/convention_definition.ncml)
