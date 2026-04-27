---
modified:
  - 2026-04-27T14:18:36+02:00
created: 2026-04-27T13:13:22+02:00
---
#Peer_Review [[2026-04-27]] #BMC_Geriatrics #Sepsis 
Reviewer Summary:

Recommendation:
Major Comments:
1. This manuscript does not present any data separated by age, e.g., in patients >60 vs <60 y.o. While the cohort contains predominantly elderly patients, no conclusion about the contribution of age to sepsis pathology can be reached without a dedicated comparison. 
2. Discussion, "The mortality prediction model was validated as high-quality and ready for clinical implementation.": The model was developed but not validated in an independent cohort. Including: Discussion, "Immunosenescence was evidenced": Neither the observed lymphopenia not hypoproteinemia can be clearly attributed to immunosenescence; especially in the absence of any comparison by age. 
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
15. Discussion, "In the later stages of infection": Many studies have explored early and concomitant septic hyperinflammation and hypoinflammation; putting into question the older concept of two-staged sepsis-pathology. Please consider discussing your results in this light. 
16. 
Language & Proofing:
17. Abstract, Methods, "TP-NLR": This abbreviation is used here but only introduced in the next paragraph. Please avoid non-standard abbreviations wherever possible and introduce abbreviations at first use.
18. Abstract, Results, "FDR-adjusted P": FDR-adjusted p-values are more commonly referred to as q-values. 
19. Starting on page 10, all pages are numbered "1". 
20. Figure 3: This figure was almost certainly produced in Biorender. Biorender requires citation before publishing any figures designed using this tool. In its current state, this figure is rather confusing. It misses a clear start and endpoint and visual flow. Please consider limiting the scope of this figure to the findings of this manuscript and improving its structure and clarity. 
Further Suggestions:
21. You may want to consider repeated subsampling to bootstrap a number of training and validation cohorts and confirm the robustness of your model. 
22. Figure 2: Consider re-graphing these showing all patients, indicating their mortality either by dot color or dot shape.

File:

Confidential notes to the editor:

Other notes:

![[Revised Manuscript.pdf]]

![[Revised Manuscript_Supplement.pdf]]