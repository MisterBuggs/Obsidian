---
modified:
  - 2026-04-28T14:57:47+02:00
created: 2026-04-28T14:56:25+02:00
---
#Random_Idea #MHH #BMC #Writing #Team #Science_Computer #Manuscript_Secondary
TLDR: A student could lead a rather simple data science project that copies the methodology of the paper below but in a target-agnostic manne

I just had an interesting idea for a data-driven paper when reading this: https://bmjoncology.bmj.com/content/5/1/e001037 . The authors suspect that TIGIT inhibitors represent a case of 'herding' in which five or more big pharma companies compete to develop essentially the same product, even though there is no proof-of-concept (in the form of an approved drug for that target yet). The authors pose that the risk does not justify the means, thereby starving other research of funding. I find it astonishing that you can publish this in BMJ Onc while only focusing on one narrow drug type. Intuitively, it should be conceptually simple (even though computationally expensive) to scrape Clinicaltrials.gov for titles, keywords, and possibly study descriptions of all ongoing clinical trials, perform a clustering analysis by keywords or targets or sth, manually (or LLM assisted) filter for studies with already approved targets, and present a much more comprehensive review of targets / mechanisms at high risk of 'herding'. Does that sound interesting for you?