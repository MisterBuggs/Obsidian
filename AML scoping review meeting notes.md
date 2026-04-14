---
modified:
  - 2026-04-14T17:25:29+02:00
created: 2026-04-14T16:12:20+02:00
---
Make a table of different columns (connected by an AND statement), with rows representing different OR-connected terms within each AND bracket. 
Consider running a Python script on all possible subset of this table. This helps you develop an intuition for the relative contribution of each search term.

Example for a table-expression of the search string below:

| AML | bone marrow | Cell therapy   | microenvironment | immune evasion |
| --- | ----------- | -------------- | ---------------- | -------------- |
| MDS |             | Immune therapy | niche            | immune escape  |
|     |             | Immunotherapy  |                  |                |
(AML OR MDS) AND (bone marrow) AND ((cell therapy) or (immune therapy) OR (immunotherapy)) AND (microenvironment OR niche) AND (evasion OR escape)

The number of possible searches from such a table without allowing a search string with only is 2<sup>(k<sub>1</sub>+k<sub>2</sub>[...]+k<sub>i</sub>)</sup>-1-SUM<sub>1,2,3,[...],i</sub>(2<sup>k<sub>i</sub></sup>-1)
where i is the number of columns and k is the number of items per column; for the table above, that would be 1024-1-17

After letting a Python script iterate over different subsets of the largest sensical table, returning titles and PMIDs found through each search string as .csv, as well as returning the numbers of papers found with each string. This gives you a hyper-space of all potentially sensical searches and their results. You can even do statistics in that space, like elbow-point of search string length vs number of results, or regress the number of results on individual columns and on individual search terms. Eventually, we can decide for the search string with the largest manageable number of results (e.g.,<500) that also makes sense and covers the positive control papers:
Select the largest manageable and sensical search string by going back and forth between searches and a list of known important papers (this serves as a positive control). The list may grow during this iterative process and the search string will be refined. 
