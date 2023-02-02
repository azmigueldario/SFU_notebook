# Meetings

## 20230202 - Matthew Nguyen

Will told me that I could aim to reproduce the approach of Matthew's analysis with _E. coli_ isolates from Qatar: situating them in a global epidemiological context. My initial plan was to use this data for training only. Now, I would like to obtain representative global sequences, and characterize all _Klebsiella spp._ isolates (Qatar and Global). Then, represent diversity in Qatar using phylogenetic trees and then a summary of how this strains relate to a global tree. 

**Suggestions:**  
- For mobile elements prediction: ME finder and MGE finder. 
- Pop and partitioning cluster tool (PopPunk). Uses core and accessory genes.  
- Once I find the location of Qatari isolates in global population, characterize the  most closely circulating ST or clonal complex and compare it with local strains.  
- Matthew  used around 10,000 isolates for **PopPUNK**:  

   - To reduce the total number use MASH and pick 100 isolates that are close to every Klebsiella spp. strain.
   - Optimize the model if it is too heterogeneous
- Aishwarya uses a tool for plasmids/mobile elements, use Plasmid finder
- Overall quality of sequencing reads obtained from the Qatar collaboration was poor. M' found several small contigs after assembly and unexpectedly small plasmids.
