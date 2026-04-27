---
modified:
  - 2026-04-27T14:08:17+02:00
created: 2026-04-27T13:13:22+02:00
---
#Peer_Review [[2026-04-27]] #BMC_Geriatrics #Sepsis 
Reviewer Summary:

Recommendation:
Major Comments:
1. This manuscript does not present any data separated by age, e.g., in patients >60 vs <60 y.o. While the cohort contains predominantly elderly patients, no conclusion about the contribution of age to sepsis pathology can be reached without a dedicated comparison. 
2. Discussion, "The mortality prediction model was validated as high-quality and ready for clinical implementation.": The model was developed but not validated in an independent cohort. 
Minor Comments:
3. Introduction: Secondary infections in bacterial sepsis have been extensively explored. Consider briefly introducing findings from other studies as they pertain to the subsets of viral secondary infections and geriatric populations.
4. Methods, Study subjects and design: Please provide tabular fine-grained exclusion criteria, e.g., what diseases were considered hematologic disorders (was any anemia included?), which rheumatic and autoimmune diseases were considered (where minor allergic conditions included, for example?), *severe* renal disease, *major* trauma, etc.
5. Methods, Study subjects and design: Obtaining written informed consent from sepsis patients is infamously hard. This is aggravated in a geriatric cohort with a higher burden of comorbidities. Was written informed consent provided by each patient? Was written informed consent from legal guardians or first-degree relatives permissible? 
6. Methods, Study variables: Please specify which mortality was considered as primary outcome. Given the data type, I assume that total hospital mortality was used and not, for example, 28 day mortality. 
7. Methods, tNGS: Were data compared with or confirmed by microbiological culture? This is still considered the Gold Standard for the sensitive and specific detection of pathogenic bacteria.
8. Table 1: Please indicate in the table description  whether these p-values were multiplicity corrected. Alternatively, consider naming the respective column q-values if these represent corrected p-values. 
9. Table 1: Please consider including common sepsis-related organ-failure scores, like qSOFA and SOFA, or the components that make up these scores. Please also consider including the Charlson Comorbidity Index in this table. 
10. Figure 1 and associated results: Please discuss the risk of overfitting, especially in the absence of a validation cohort. NLR is an aggregated result. Do you observe similar results when looking at ```TP x Neutrophil count x Lymphocyte count```?
11. Results, Multivariates logistic analysis: TP and NLR the independent factors impacting survival status, "The table presents paired values...": Which table?
12. Results, Multivariates logistic analysis: TP and NLR the independent factors impacting survival status, "The simultaneous presence of lymphopenia, neutrophilia, and hypoproteinemia reflects...": Does this represent an interpretation or a literature review? In the former case, the data presented up to this point may tentatively but certainly not definitively support this conclusion. If this is meant to place the findings in the context of literature, please add citations. In either case, this may be better placed in the discussion section. 
13. Table 2: According to page 13, this table represents patients positive or negative for viral co-infections. According to page 15, the table represents patients positive / negative for bacteria, fungi, AND viruses. Please clarify in the text, the table title, and the table description, which of these is true. 
14. Please make sure to discuss the limitation that your data cannot discern cause and effect. It is as plausible that viral infection further worsens pathology as it is plausible that more severely septic patients have a higher risk of viral infection or endogenous viral reactivation. 
15. 
Language & Proofing:
16. Abstract, Methods, "TP-NLR": This abbreviation is used here but only introduced in the next paragraph. Please avoid non-standard abbreviations wherever possible and introduce abbreviations at first use.
17. Abstract, Results, "FDR-adjusted P": FDR-adjusted p-values are more commonly referred to as q-values. 
18. Starting on page 10, all pages are numbered "1". 
19. Figure 3: This figure was produced in biorender. 
Further Suggestions:
20. You may want to consider repeated subsampling to bootstrap a number of training and validation cohorts and confirm the robustness of your model. 
21. Figure 2: Consider re-graphing these showing all patients, indicating their mortality either by dot color or dot shape.

File:

Confidential notes to the editor:

Other notes:

![[Revised Manuscript.pdf]]

![[Revised Manuscript_Supplement.pdf]]