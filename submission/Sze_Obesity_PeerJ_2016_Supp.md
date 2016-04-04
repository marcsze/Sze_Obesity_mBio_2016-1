# Bacterial Microbiome and Obesity Meta-analysis Supplemental
Marc Sze  
January 26, 2016  
<br><br>





###In-Depth Overview of Search Strategy:

The inital search strategy included looking for all papers that initially fit under the below NCBI PubMed advanced search criteria.  The terms included in this criteria were that the manuscript had to have "Bacterial Microbiome" and "Obesity, BMI, bmi, obesity" in their manuscript criteria, it was not published more than 10 years ago, they were not review articles, and it contained research on humans only.  The below formula when put into PubMed should recapitulate our initial search on the website.


```bash
(((((((((Bacterial Microbiome) AND (Obesity or bmi or body mass index or BMI or obesity) AND "last 10 years"[PDat] AND Humans[Mesh])) NOT review[ptyp]) AND "last 10 years"[PDat] AND Humans[Mesh])) AND "last 10 years"[PDat] AND Humans[Mesh])) AND "last 10 years"[PDat] AND Humans[Mesh])
```

This search yielded a total of 187 manuscripts.  From two previous other reviews of obesity and the bacterial microbiome along with knowledge of two other published papers that investigated obesity but were missed by the database search we obtained a total of 7 more articles.  We also had access to normal healthy individuals from an unpublished dataset.  This brought our total number of records to 196.  

From this total we browsed abstracts for mention of stool or feces examination, that did not involve children, was not a clinical trial for probiotics or other diet related treatments, did not only have participants with inflammatory bowel disease, the articles were in English, did not only use PCR, qPCR, or RT-PCR only for their analysis, and sequecning that used only clone libraries.  This ultimately excluded all but a total of 11 studies.

From this total of 12 studies the full text was reviewed for whether or not sequencing data was publicly available, BMI information (either categorical or continuous) was available in a supplement or, if it was not available, whether authors upon contact were willing to share this information or direct us to repositories that stored this specific information.  One study was excluded [@yatsunenko_human_2012] because it contained children and their sequencing of the 16S rRNA gene involved amplicons of only 100bp in length.  They also did not have obesity as part of their results in the actual published manuscript.  The second study was excluded because they did not have BMI information available and when contacted the authors never returned any correspondance [@zeevi_personalized_2015].  A thrid study was excluded since it did not use 16S rRNA gene sequencing for their bacterial microbiome analysis [@arumugam_enterotypes_2011]  

Once these 3 studies were excluded there was a total of 9 studies in the qualitative synthesis of the analysis.  Because we decided a prioi to use the standard definition for BMI group classification one study from this ten did not have any individuals who were obese by this criteria [@nam_comparative_2011] and was excluded from the final quantitative synthesis and analysis.  

*Inclusion Criteria:*

* Contains mention of Bacterial Microbiome and Obesity
* BMI, bmi, or obesity could be referenced instead of Obesity
* Not published more than 10 years ago
* Research on Humans only
* At least one specific result examining obesity and a bacterial microbiome measure
* Participants did not have Inflammatory Bowel Disease or Cancer
* Greater than 100bp single or dual end reads for 16S Sequencing
* DNA obtained from stool or feces

*Exclusion Criteria:*

* PCR, qPCR, metagenomic sequencing, or RT-PCR used as main analysis Tool
* TRFLP or clone sequencing used to asses the bacterial community
* Utilization of 100bp or less single end reads for sequencing
* Sequencing Data not publicly available for download
* BMI not available and authors do not return correspondance
* Samples were not stool or feces
* Study contained children
* Study was a review





###Supplemental Results:


**Table S1. Summary of P-values for Measurements of Interest for each Individual Study for Obese versus Normal**


|    Study    |  Bacteroidetes  |  Firmicutes  |  B/F Ratio  |  Shannon Diversity  |  OTU Richness  |  Evenness  |  Bray Curtis  |
|:-----------:|:---------------:|:------------:|:-----------:|:-------------------:|:--------------:|:----------:|:-------------:|
|   Baxter    |      0.20       |     0.61     |    0.19     |        0.07         |      0.04      |    0.13    |     0.06      |
|    Ross     |      0.20       |     0.38     |    0.22     |        0.28         |      0.25      |    0.38    |     0.82      |
|  Goodrich   |      0.44       |     0.29     |    0.27     |        0.61         |      0.32      |    0.84    |     0.33      |
|   Escobar   |      0.06       |     0.13     |    0.08     |        0.95         |      0.23      |    0.62    |     0.07      |
|  Zupancic   |      0.57       |     0.60     |    0.59     |        0.31         |      0.16      |    0.44    |     0.18      |
|     HMP     |      0.46       |     0.79     |    0.59     |        0.58         |      0.97      |    0.47    |     0.81      |
|     Wu      |      0.71       |     0.75     |    0.99     |        0.91         |      0.28      |    0.38    |     0.53      |
|  Turnbaugh  |      0.57       |     0.94     |    0.62     |        0.97         |      0.54      |    0.88    |     0.75      |



![Figure S1](results/figures/suppFigClostridiales.png)\


**Figure S1: Boxplots of All Clostridiales OTUs Important for Non-Obese and Obese Classification.** A total of 6/8 studies had at least 1 OTU that was classified to Clostridiales.  However, overall there was a wide range in direction both between studies and within studies.


![Figure S2](results/figures/suppFigLachno.png)\


**Figure S2: Boxplots of All Lachnospiraceae OTUs Important for Non-Obese and Obese Classification.** A total of 6/8 studies had at least 1 OTU that was classified to Lachnospiraceae.  However, overall there was a wide range in direction both between studies and within studies.


![Figure S3](results/figures/suppFigRumi.png)\

**Figure S3: Boxplots of All Ruminococcaceae OTUs Important for Non-Obese and Obese Classification.** A total of 7/8 studies had at least 1 OTU that was classified to Ruminococcaceae  However, overall there was a wide range in direction both between studies and within studies.


**Table 2. Summary of BLAST alignment of Representative OTU for Clostridiales, Lachnospiraceae, and Ruminococcaceae classification OTUs**  


|    Study    |  Hypervariable Region  |                                                 Clostridiales OTU Most Prevalent Match                                                 |                                                                                                    Clostridiales OTU Highest Match (Query Cover&#124;Identity)                                                                                                    |                                              Lachnospiraceae OTU Most Prevalent Match                                               |                                                                                                                    Lachnospiraceae OTU Highest Match (Query Cover&#124;Identity)                                                                                                                     |                                                                               Ruminococcaceae OTU Most Prevalent Match                                                                                |                                                                                                                                                                Ruminococcaceae OTU Highest Match (Query Cover&#124;Identity)                                                                                                                                                                 |
|:-----------:|:----------------------:|:--------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|     Wu      |         V1-V2          |                                                          OTU191:3 Clostridium                                                          |                                                                                                                Anaerovorax odorimutans(81&#124;95)                                                                                                                |                            OTU75:6 Clostridium &#124; OTU229:5 Peptostreptococcus &#124; OTU5: 4 Blautia                            |                                                                                                                                   Clostridium bolteae(100&#124;96)                                                                                                                                   |                                                                                         OTU123:4 Clostridium                                                                                          |                                                                                                                                                                            Anaerotruncus colihominis(85&#124;95)                                                                                                                                                                             |
|  Turnbaugh  |           V2           |                                                                  None                                                                  |                                                                                                                               None                                                                                                                                |                                                               OTU318:                                                               |                                                                                                                                                                                                                                                                                                      |                                                                                                OTU30:                                                                                                 |                                                                                                                                                                                                                                                                                                                                                                                              |
|    Ross     |         V1-V3          |                                                         OTU102:8 Ruminococcus                                                          |                                                                                                            Ruminococcus champanellensis(100&#124;100)                                                                                                             |                                                         OTU177:3 Roseburia                                                          |                                                                                                                                     Clostridium Spp(100&#124;98)                                                                                                                                     |                                                                                         OTU22:3 Oscillibacter                                                                                         |                                                                                                                                                                                Oscillibacter Spp(100&#124;97)                                                                                                                                                                                |
|   Escobar   |         V1-V3          |                                                                  None                                                                  |                                                                                                                               None                                                                                                                                |                                  OTU67:6 Eubacterium &#124; OTU74:5 Blautia &#124; OTU80:4 Blautia                                  |                                                                                      OTU67:Dorea formicigenerans(100&#124;96) &#124; OTU74:Blautia luti(100&#124;96) &#124; OTU80:Blautia producta(100&#124;93)                                                                                      |                                                                                                 None                                                                                                  |                                                                                                                                                                                             None                                                                                                                                                                                             |
|  Zupancic   |         V1-V3          |                               OTU322:3 Parvimonas &#124; OTU62:3 Parvimonas &#124; OTU256:4 Clostridium                                |                                                              OTU322:Eubacterium hallii(92&#124;100) &#124; OTU62:Eubacterium hallii(92&#124;99) &#124; OTU256:Romboutsia lituseburensis(92&#124;99)                                                               |                                                                None                                                                 |                                                                                                                                                 None                                                                                                                                                 |                                                           OTU398:3 Eubacterium &#124; OTU850:3 Ethanoligenens &#124; OTU817:8 Ruminococcus                                                            |                                                                                                                     OTU398:Oscillibacter valericigene(77&#124;96) &#124; OTU850:Papillibacter cinnamivorans(90&#124;91) &#124; OTU817:Ruminococcus callidus(91&#124;95)                                                                                                                      |
|   Baxter    |           V4           |                                                         OTU479:4 Anaerostipes                                                          |                                                                                                               Anaerosporbacter mobilis(100&#124;94)                                                                                                               |                                                         OTU36:8 Clostridium                                                         |                                                                                                                                  Lachnobacterium bovis(100&#124;96)                                                                                                                                  |                                                                                         OTU320:4 Clostridium                                                                                          |                                                                                                                                                                             Sporobacter termitidis(100&#124;94)                                                                                                                                                                              |
|  Goodrich   |           V4           |  OTU287:5 Clostridium &#124; OTU662:8 Clostridium &#124; OTU145:6 Clostridium &#124; OTU244:3 Clostridium &#124; OTU442:4 Clostridium  |  OTU287:Clostridium fimetarium(100&#124;94) &#124;  OTU662:Ruminiclostridium thermocellum(100&#124;91) &#124; OTU145:Ruminococcus lactaris(100&#124;100) &#124; OTU244:Anaerosporobacter mobilis(100&#124;94) &#124; OTU442:Clostridium propionicum(100&#124;92)  |  OTU180:4 Clostridium &#124; OTU75: 5 Clostridium &#124; OTU122:5 Roseburia &#124; OTU92:8 Clostridium &#124; OTU673:7 Clostridium  |  OTU180:Hungatella effluvii(100&#124;95) &#124; OTU75:Ruminococcus gnavus(100&#124;100) &#124; OTU122:Eubacterium ramulus(100&#124;96) &#124; OTU92:Fusicatenibacter saccharivorans(100&#124;94) &#124; OTU673:Bacteroides xylanolyticus(92&#124;97) &#124; OTU523:Eubacterium rectale(100&#124;97)  |  OTU68:5 Ruminococcus &#124; OTU255:3 Blautia &#124; OTU1145:3 Ethanoligenens &#124; OTU558:8 Ruminococcus &#124; OTU297:3 Oscillibacter &#124; OTU6:4 Clostridium &#124; OTU132:2 Ruminoclostridium  |  OTU68:Acetanaerobacterium elongatum(100&#124;89) &#124; OTU255:Pseudoflaconifractor capillosus(100&#124;91) &#124; OTU1145:Subdoligranulum variable(100&#124;95) &#124; OTU558:Ruminococcus champanellensis(92&#124;98) &#124; OTU297:Flavonifractor plautii(100&#124;95) &#124; OTU6:Oscillibacter valericigenes(100&#124;93) &#124; OTU132:Intestinimonas butyriciproducens(100&#124;92)  |
|     HMP     |         V3-V5          |                                                          OTU293:7 Clostridium                                                          |                                                                                                            Anaerobacterium chartisolvens (100&#124;91)                                                                                                            |                                                                None                                                                 |                                                                                                                                                 None                                                                                                                                                 |                                                                                         OTU252:10 Clostridium                                                                                         |                                                                                                                                                                                 Clostridium Spp(100&#124;97)                                                                                                                                                                                 |


**Table S3. Top Sequence Similarity by Variable Region Based on the Representative OTU for Clostridiales, Lachnospiraceae, and Ruminococcaceae**  


|  Variable Region  |  Top Similarity Colstridiales (Id&#124;Gaps)  |  Average Similarity Colstridiales (Id&#124;Gaps)  |  Top Similarity Lachnospiraceae (Id&#124;Gaps)  |  Average Similarity Lachnospiraceae (Id&#124;Gaps)  |  Top Similarity Ruminococcaceae (Id&#124;Gaps)  |  Average Similarity Ruminococcaceae (Id&#124;Gaps)  |
|:-----------------:|:---------------------------------------------:|:-------------------------------------------------:|:-----------------------------------------------:|:---------------------------------------------------:|:-----------------------------------------------:|:---------------------------------------------------:|
|    V1-V2 vs V2    |                     None                      |                       None                        |                       TBC                       |                         TBC                         |                       TBC                       |                         TBC                         |
|       V1-V3       |                50.3&#124;39.1                 |                  48.0&#124;42.4                   |                 40.1&#124;47.9                  |                   35.9&#124;52.5                    |                 30.4&#124;68.4                  |                   29.2&#124;69.1                    |
|        V4         |                 89.4&#124;2.8                 |                   87.7&#124;2.4                   |                  92.9&#124;0.8                  |                    85.8&#124;5.6                    |                  86.6&#124;0.8                  |                    81.6&#124;6.6                    |


![](results/figures/Figure_S4-1.png)\

**Figure S4: Funnel Plot of the Shannon Diversity and Bacteroidetes/Firmicutes Ratio Relative Risk.**  **A)** Overall there does not seem to be any bias associated with the studies selected for the Shannon Diversity analysis with studies falling on either side of the predicted value and those with smaller total n falling further away from this. **B)** For the Bacteroidetes/Firmicutes ratio there does not seem to be any associated bias. The overall pattern is similar to that observed for the Shannon Diversity analysis.


![](results/figures/Figure_S5-1.png)\

**Figure S5. Summary of Power and Sample Size Simulations for OTU Richness (S) for Non-obese versus Obese**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.

![](results/figures/Figure_S6-1.png)\

**Figure S6. Summary of Power and Sample Size Simulations for Evenness (J) for Non-obese versus Obese**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.


![](results/figures/Figure_S7-1.png)\

**Figure S7. Summary of Power and Sample Size Simulations for Bacteroidetes for Non-obese versus Obese**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.

![](results/figures/Figure_S8-1.png)\

**Figure S8. Summary of Power and Sample Size Simulations for Firmicutes for Non-obese versus Obese**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.


![](results/figures/Figure_S9-1.png)\

**Figure S9. Summary of Power and Sample Size Simulations for Relative Risk  for Obesity based on Shannon Diversity**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.

![](results/figures/Figure_S10-1.png)\

**Figure S10. Summary of Power and Sample Size Simulations for Relative Risk  for Obesity based on B/F Ratio**  P-values listed in the legend represent the outcome of a wilcoxson rank sum test between non-obese and obese for each specific data set.  **A)** The dotted lines represent the actual effect size for each specific study.  **B)** The dotted lines represent the needed n to achieve an 80% power with the actual study effect size.


*******
##### References:
