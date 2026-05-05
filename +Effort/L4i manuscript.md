---
modified:
  - 2026-04-01T11:44:56+02:00
  - 2026-04-09T17:05:24+02:00
  - 2026-04-14T15:45:26+02:00
  - 2026-04-15T17:15:50+02:00
  - 2026-04-16T17:48:10+02:00
  - 2026-04-17T16:48:33+02:00
  - 2026-04-22T16:38:46+02:00
  - 2026-04-24T15:25:52+02:00
  - 2026-04-28T12:56:57+02:00
  - 2026-05-05T16:53:45+02:00
created: 2026-03-16T15:31:22+01:00
tags:
  - JHU
  - Writing
---
# Header
Target Journal: **Blood**
Author list, **not confirmed**: WB, #People_SDV , #People_SGD, #People_MoBe, #People_WiLi , #People_LuZi , #People_ETZ 

# Figures
[L4i manuscript_Figures_WB.pptx](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\L4i manuscript_Figures_WB.pptx>)
[[ash_bld_DigitalArtGuidelines_2020.pdf|Blood Style Guide (PDF)]]

# L4i To Do
#To_Do [[+Calendar/2026-04-09|2026-04-09]]
- [ ] WB: Statistical Hypothesis testing
	- [ ] Consider more complex tests down the line, which can integrate another axis, e.g., E8 / TIRN in ng / wb *or* E8 / TIRN for cfu-gm / cfu-e. 
	- [x] Collect all data that should undergo statistical hypothesis testing, i.e., all with 3 or more samples, in the same excel file in a format conducive to testing in R or in Python. #To_Do What library will I use? What's the preferred format for the data?
- [ ] WB: Generally use FACS plot axes in the style #People_SDV uses in [L4i manuscript_Figures_WB.pptx](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\L4i manuscript_Figures_WB.pptx>) with markers on the end of a line.
- [ ] Confirm that all plots use SEM or all SD, no back and forth.
- [ ] Attempt statistical hypothesis tests wherever possible to confirm better performance of L4i.
- [ ] WB: Attempt absolute quantifications as stacked area plots like #People_SDV 
- [ ] Make Biorender overview cartoon to tie down the story, then order figures accordingly.
- [ ] Re-analyse [HD19](app://obsidian.md/E32C4#^4550c0)?
- [ ] WB / #People_SDV : Discuss taking the d9 timepoint out of  #People_SDV 's percentage plots (yellow / green / blue), as d9 is missing from the Ng protocol data. The official Ng protocol should be measured d13-15; reviewers will criticize that we compare as early as d10-12 instead and do not show late timepoints. Transferring Ng-floaters into terminal diff media until d17  20 would really complement these data.
- [ ] #People_SDV: make Sankey-like yellow/ green/ blue percentage charts accessible in Powerpoint. 
- [x] WB / #People_SDV : Ask Elias whether to name 3 boxes by marker or by derived identity
- [x] WB: Make Sankey-like yellow/green/blue percentage charts for #People_SDV data.
- [x] WB: HD experiment in 3D. [HD16](app://obsidian.md/E32C4#^6e78e6). Match figures from [WBuys Journal Club 2026Jan15.pptx](https://1drv.ms/p/c/2c6e7480743f438e/IQAY4O5XRD3mR4zXrau-5s0EATWw3Cun9dk-1qGxBFb_ckA?e=u2CmJE) with [WB HD16.wsp](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\HD16\WB HD16.wsp>) **[specifically this file]** to confirm putative tube-naming in [Notes HD16](https://1drv.ms/t/c/2c6e7480743f438e/IQDBhbof6zDwQ4GKafsPaZu9AYjM4UkoNV0RZv7p7FdKw34?e=HHRfrp).
- [x] Add post-CFC FACS for [HD16 CFC day 11.wsp](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\HD16\HD16 CFC day 11.wsp>) to show that all cell types are there. Also for the data provided by #People_SDV . Into Suppl?
- [x] Calculate absolute production efficiencies for HD16: This file has the absolute viable numbers: [Notes HD16.txt](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\HD16\Notes HD16.txt>). This file has the percentages: [3 lines HD summary for L4i manuscript.xlsx](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\3 lines HD summary for L4i manuscript.xlsx>). 
- [x] WB: Overview teratoma: Which lines do I have imaged and counted already? Presumably requires #People_SDV or #People_WiLi to [Get files](app://obsidian.md/Get%20files): Look through teratoma boxes. I believe there should be teratoma for at least 3 iPSC lines.
- [x] WB: Add existing CFC quantifications.
- [x] WB: scour older powerpoint presentations for good HD / teratoma / chimera figures.
- [x] WB: Ask Ludovic whether he has any other E8 vs L4i teratomata.
- [x] #People_SDV : Send 1) percentages CD34 / 45 quadrants for experiments of interest, 2) their viability in FACS, 3)Trypan-blue negative (viable) cell count at harvest, 4) original iPSC-number in that culture, 5) volume at each feeding step.

# Process notes
- No statistical significance in simple 1:1 group comparisons of efficiency using Wilcoxon matched pairs (or paired t-tests for that matter). Assuming high variance between but not within groups, i.e., L4i is always better than E8 but the extent to which a certain cell line "performs" varies wildly, I will need data for an **additional 2-3 cell lines to achieve statistical significance** for the naive efficiency calculations in Wilcoxon, as per a naive data simulation with maximum independence between lines ([paired wilcoxon test.xlsx](<file:///C:\Users\buyswill\OneDrive - mh-hannover.de\L4i manuscript\Statistical Hypothesis testing>)). Maybe #People_SDV 
. 
- Present [[+Effort/E32C6|E32C6]] HD1 for non-permissive lines, [[+Effort/JHU/E32C|E32C4]] for insufficient lineage commitment in E8, and [[E5C3 dTomato]] for a good line that got even higher numbers in L4i.
- Present [[E5C3 dTomato]] functionality.
- Frame days 12-13, 16-18, 20 as GMP, Myeloblast, Myelocyte to enable presenting d6 alongside d7 protocols.
- Show [[E32C4]] E8 vs L4i HD fully 3D using WB growth factors by SDV.
- Suppl: 
	- #People_SDV 3D HDNg E8 vs L4i.
	- [[+Effort/E32C6#^54e3bf|HD3]] media, density optimization, together with HD16 SPELS vs SPELS-BSA. 34/45/BB9 Kinetic? Also in [[+Effort/E32C6#^3c1a3e|HD9]]. 
	- WB [[E32C4#^6e78e6|HD16]] 3D HDWB E8 vs L4i disaggregated EB and quantitative CFC from disaggregated EB. Not enough cells for FACS from floaters before CFC. Possibly show results from floating cell CFC with FACS after but not before CFC.
	- WB [[E32C4#^8b6950|HD18]] composition of EBs together with CD34 fluorescence microscopy [BM organoids](https://1drv.ms/f/c/2c6e7480743f438e/IgBUIQvfYDuBRbGz6dZk6FsGAb1bwiNWQlWSfw0aQPvlQ8c?e=4pquww) (and possibly CD45; @"Malika Sharma" <msharm50@jh.edu>).
	- MD11HD with [[CFC]].
	- [[Adherent]] cell [[Flow Cytometry]], e.g., [[E32C4]] HD12 [[E8]] vs [[L4i]] into the supplement?
	- If data is present in a matched E8 vs L4i manner, e.g., in  [[E32C4#^6e78e6|HD16]], show that these are really just BB9+ hemangioblast (potentially questioning their true HSC status but making them super potent for myeloid cell production)
	- ![[Meeting SDV|Meeting 2026-04-24]]