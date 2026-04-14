---
modified:
  - 2026-04-14T16:52:00+02:00
created: 2026-04-14T16:12:20+02:00
---
Make a table of different columns (connected by an AND statement), with rows representing different OR-connected terms within each AND bracket. 
Consider running a Python script on all possible subset of this table. This helps you develop an intuition for the relative contribution of each search term.
Select the largest manageable and sensical search string by going back and forth between searches and a list of known important papers (this serves as a positive control). The list may grow during this iterative process and the search string will be refined. 
Example for a table-expression of a search string:

| AML | bone marrow | Cell therapy   | microenvironment | immune evasion |
| --- | ----------- | -------------- | ---------------- | -------------- |
| MDS |             | Immune therapy | niche            | immune escape  |
|     |             | Immunotherapy  |                  |                |
(AML OR MDS) AND (bone marrow) AND ((cell therapy) or (immune therapy) OR (immunotherapy)) AND (microenvironment OR niche) AND (evasion OR escape)

| AML                      | bone marrow | Cell therapy   | microenvironment | crosstalk   |     |
| ------------------------ | ----------- | -------------- | ---------------- | ----------- | --- |
| MDS                      |             | Immune therapy | niche            | interaction |     |
| acute myeloid leukemia   |             | Immunotherapy  |                  |             |     |
| myelodysplastic syndrome |             |                |                  |             |     |
