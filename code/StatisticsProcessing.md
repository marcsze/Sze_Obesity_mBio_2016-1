# Bacterial Microbiome and Obesity Meta-analysis Statistics Pipeline
Marc Sze  
January 29, 2016  
<br>

#### **Preamble**  
The following workflow was used to generate all the statistical analysis of our meta-analysis.  It is organized by part of which there are three.  Code chunks will be displayed as such below:


```r
print("Hello World!")
x = 1 + 1
print(x)
```
Data tables are provided in this repository for your use if you do not want to have to repeat the somewhat laborious sequence analysis pipeline from start to finish.  BMI data if publicly available has been kept in this repository for your ease of reference.  However, there are a few data sets in which the BMI data is not public and as such you will need to request this information from the respective sources if you would like to use them.  The two data sets this mostly concerns are the Wu, et al study and the HMP study.
<br><br>


#### **Part 1: Seperate Analysis of each Study**  
The statistical analysis will follow a general order of data loading, alpha diversity and BMI analysis, PERMANOVA of Bray-Curtis distance matrix on obese versus non-obese and BMI classifications (where available), relative risk, and then classification using AUCRF.
<br><br>

##### **Baxter et al. 2016:**  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

# Reading in the necessary Data
setwd("C:/Users/marc/Desktop/obesity2/data1")
demographics <- read.csv("demographics.v2.csv")
microbiome <- read.csv("data1.subsample.otus.csv")
phylogenetic.info <- read.csv("data1.summary.taxonomy.csv")
taxonomy <- read.csv("data1.taxonomy.csv")

# Minor modifications and Rownames adustments of data tables
rownames(demographics) <- demographics[,1]
rownames(microbiome) <- microbiome[,2]
demographics <- demographics[,-1]
microbiome <- microbiome[,-2]
test.samples <- rownames(demographics)
microbiome <- microbiome[,-c(1:2)]

# Create Table for phyla information
phylogenetic.info <- phylogenetic.info[-c(1:12),]
phyla <- which(phylogenetic.info$taxlevel == 2)
phyla.table <- phylogenetic.info[phyla, ]
phyla.table <- phyla.table[,-c(1:2,4:5)]
phyla.names <- as.character(phyla.table[,1])
phyla.table <- phyla.table[,-1]
phyla.table <- as.data.frame(t(phyla.table))
colnames(phyla.table) <- phyla.names
rownames(phyla.table) <- rownames(demographics)


#Add phyla together that are not very abundant and delete them from the table
phyla.table$other <- apply(phyla.table[, c("Acidobacteria", "Deferribacteres", "Deinococcus-Thermus", "Fusobacteria", "Lentisphaerae", "Spirochaetes", "Synergistetes", "Tenericutes", "Cyanobacteria_Chloroplast", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(1, 4:7, 9:11, 13, 15)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
phyla.table.rel.abund <- phyla.table.rel.abund[, -8]

#Generate Alpha Diversity Measures and alpha diversity table
library(vegan)
H <- diversity(microbiome)
S <- specnumber(microbiome)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)

#Create age quartiles

demographics$age.quartile <- with(demographics, cut(Age, breaks=quantile(Age, probs=seq(0,1, by=0.25)), include.lowest=TRUE))

#Create Obese Yes/No groups
demographics$obese[demographics$BMI.classification=="Normal" | demographics$BMI.classification=="Overweight"] <- "No"
demographics$obese[demographics$BMI.classification=="Obese" | demographics$BMI.classification=="Extreme Obesity"] <- "Yes"

#Create Obese.num groups
demographics$obese.num[demographics$obese=="No"] <- 0
demographics$obese.num[demographics$obese=="Yes"] <- 1
obese <- factor(demographics$obese.num)

######################################################################################## First Level Analysis & Alpha Diversity with BMI #############
###########################################################################

#Create a column with obese and extreme obese as single entity
demographics$BMIclass2[demographics$BMI.classification=="Normal"] <- "Normal"
demographics$BMIclass2[demographics$BMI.classification=="Overweight"] <- "Overweight"                     
demographics$BMIclass2[demographics$BMI.classification=="Obese" | demographics$BMI.classification=="Extreme Obesity"] <- "Obese"


##Test BMI versus alpha diversity and phyla

bmi <- demographics$BMI
bmi.class <- factor(demographics$BMI.classification)
bmi.class2 <- factor(demographics$BMIclass2)

anova(lm(H ~ obese)) #P-value=0.01643
anova(lm(H ~ bmi.class)) #P-value=0.06
anova(lm(H ~ bmi.class2)) #P-value=0.04471
a2 <- aov(H ~ bmi.class2)
TukeyHSD(x=a2, 'bmi.class2', conf.level=0.95)
# Obese-Norm (p-value=0.201), Over-Norm (p-value=0.769), Over-Obese (p-value=0.036)

anova(lm(S ~ obese)) #P-value=0.01571
anova(lm(S ~ bmi.class)) #P-value=0.06
anova(lm(S ~ bmi.class2)) #P-value=0.03
a2 <- aov(S ~ bmi.class2)
TukeyHSD(x=a2, 'bmi.class2', conf.level=0.95)
# Obese-Norm (p-value=0.025), Over-Norm (p-value=0.546), Over-Obese (p-value=0.178)

anova(lm(J ~ obese)) #P-value=0.03241
anova(lm(J ~ bmi.class)) #P-value=0.0272
a2 <- aov(J ~ bmi.class)
TukeyHSD(x=a2, 'bmi.class', conf.level=0.95)
# Obese-Norm (p-value=0.02), Over-Norm (p-value=0.047), Over-Obese (p-value=0.06)
anova(lm(J ~ bmi.class2)) #P-value=0.02
a2 <- aov(J ~ bmi.class2)
TukeyHSD(x=a2, 'bmi.class2', conf.level=0.95)
# Obese-Norm (p-value=0.630), Over-Norm (p-value=0.159), Over-Obese (p-value=0.019)

##linear correlations
H.bmi.test <- lm(H ~ bmi)
summary(H.bmi.test)  
#R2 = 0.01246, P-value=0.0774

S.bmi.test <- lm(S ~ bmi)
summary(S.bmi.test)
#R2 = 0.03222, P-value=0.0105

J.bmi.test <- lm(J ~ bmi)
summary(J.bmi.test)
#R2 = -0.0003052, P-value=0.332

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.3175
anova(lm(bacter ~ bmi.class)) #P-value=0.6457
anova(lm(bacter ~ bmi.class2)) #P-value=0.5993

anova(lm(firm ~ obese)) #P-value=0.6817
anova(lm(firm ~ bmi.class)) #P-value=0.9293
anova(lm(firm ~ bmi.class2)) #P-value=0.8936

anova(lm(BFratio ~ obese)) #P-value=0.6305
anova(lm(BFratio ~ bmi.class)) #P-value=0.8334
anova(lm(BFratio ~ bmi.class2)) #P-value=0.881

###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################


table(demographics[,23])
###Normal=54, Overweight=71, Obese=45, Extreme Obesity=2
###Need to omit extreme obesity
#Removing rows with extreme obesity
test <- which(demographics$BMI.classification == "Extreme Obesity")
BMI.class.demo <- demographics[-test,]
BMI.microbiome <- microbiome[-test,]
bmi.class.group <- factor(BMI.class.demo$BMI.classification)

set.seed(3)
adonis(BMI.microbiome ~ bmi.class.group, permutations=1000)
#Not significant PERMANOVA = 0.2278, pseudo-F = 1.1323

set.seed(3)
adonis(microbiome ~ bmi.class, permutations=1000)
#Not significant PERMANOVA = 0.1848, pseudo-F = 1.1368

set.seed(3)
adonis(microbiome ~ bmi.class2, permutations=1000)
#Not significant PERMANOVA = 0.1359, pseudo-F = 1.1937

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe

#Shannon diversity
median(H) # 3.67469
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 198
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.692766
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Run the RR for Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:86), 4])
table(test3[c(87:172), 4])
#Group1 (Higher than median), obese = 22 and non-obese = 64
#Group2 (Lower than median), obese = 25 and non-obese = 61

group1 <- c(22, 64)
group2 <- c(25, 61)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.14
## CI = 0.70, 1.85
## p-value = 0.608

##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.3621609
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:86), 2]) #higher group
table(test4[c(87:172), 2]) #lower group
#Group1 (Higher than median), obese = 23 and non-obese = 63
#Group2 (Lower than median), obese = 24 and non-obese = 62

group1 <- c(23, 63)
group2 <- c(24, 62)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.04
## CI = 0.64, 1.70
## p-value = 0.864

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#generate test set
  # get rid of values that only have 0 and something else
testset <- Filter(function(x)(length(unique(x))>2), microbiome)
  # get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), microbiome)

#Need to add phyla and alpha diversity measures to the dataset

testset <- cbind(testset, H, S, J, phyla.table.rel.abund)
testset <- cbind(obese, testset)
         
#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 14 Measures, AUCopt = 0.7553
plot(fit)
```
<br><br>

##### **Ross et al. 2015:**  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

setwd("C:/users/marc/Desktop/obesity2/hispanic/")

hispanic.microb <- read.table("hispanic.subsample.shared", header=T)

metadata <- read.csv("s40168-015-0072-y-s1.csv")
sample.match <- read.csv("Hispanic_dataset.csv")

#Create a microbiome data table with sample names from metadata
test <- cbind(as.character(sample.match$Run_s), as.character(sample.match$Library_Name_s))
test <- test[order(test[, 1]), ]

test2 <- cbind(test, hispanic.microb)
rownames(test2) <- test2[, 2]
his.microb.edit <- test2[, -c(1:5)]
edit.metadata <- metadata[, -c(2:5)]
rownames(edit.metadata) <- edit.metadata[, 1]
edit.metadata <- edit.metadata[, -1]

#Create a metadata file in the order of the microbiome data
order1 <- rownames(his.microb.edit)
edit.metadata2 <- edit.metadata[order1, ]

#Get alpha diversity of the samples
library(vegan)

H <- diversity(his.microb.edit)
S <- specnumber(his.microb.edit)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(his.microb.edit)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- his.microb.edit
colnames(phyla.table) <- phyla.names
  #add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)
  #combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Deinococcus-Thermus", "Elusimicrobia", "Fusobacteria", "Lentisphaerae", "Synergistetes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(3:4, 6:7, 9:10)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
phyla.table.rel.abund <- as.data.frame(phyla.table.rel.abund[, -8])


#Create BMI groups
edit.metadata2$BMI.class[edit.metadata2$BMI<=24] <- "Normal"
edit.metadata2$BMI.class[edit.metadata2$BMI>24 & edit.metadata2$BMI<30] <- "Overweight"
edit.metadata2$BMI.class[edit.metadata2$BMI>=30 & edit.metadata2$BMI<40] <- "Obese"
edit.metadata2$BMI.class[edit.metadata2$BMI>=40] <- "Extreme Obesity"


#Create Obese Yes/No groups
edit.metadata2$obese[edit.metadata2$BMI.class=="Normal" | edit.metadata2$BMI.class=="Overweight"] <- "No"
edit.metadata2$obese[edit.metadata2$BMI.class=="Obese" | edit.metadata2$BMI.class=="Extreme Obesity"] <- "Yes"

#Create BMI class 2 groups
edit.metadata2$BMI.class2[edit.metadata2$BMI<=24] <- "Normal"
edit.metadata2$BMI.class2[edit.metadata2$BMI>24 & edit.metadata2$BMI<30] <- "Overweight"
edit.metadata2$BMI.class2[edit.metadata2$BMI>=30] <- "Obese"

#Get paitent demographics to be tested

bmi <- edit.metadata2$BMI
obese <- factor(edit.metadata2$obese)
bmi.class <- factor(edit.metadata2$BMI.class)
bmi.class2 <- factor(edit.metadata2$BMI.class2)


######################################################################################## First Level Analysis & Alpha Diversity with BMI #############
###########################################################################

##Test BMI versus alpha diversity and phyla


anova(lm(H ~ obese)) #P-value=0.1657
anova(lm(H ~ bmi.class)) #P-value=0.5448
anova(lm(H ~ bmi.class2)) #P-value=0.3615

anova(lm(S ~ obese)) #P-value=0.1841
anova(lm(S ~ bmi.class)) #P-value=0.61
anova(lm(S ~ bmi.class2)) #P-value=0.4

anova(lm(J ~ obese)) #P-value=0.2538
anova(lm(J ~ bmi.class)) #P-value=0.6431
anova(lm(J ~ bmi.class2)) #P-value=0.492

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.343, R2=0.01476
summary(lm(S ~ bmi)) #P-value=0.345, R2=0.01466
summary(lm(J ~ bmi)) #P-value=0.472, R2=0.008502

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.2092
anova(lm(bacter ~ bmi.class)) #P-value=0.5436
anova(lm(bacter ~ bmi.class2)) #P-value=0.4374

anova(lm(firm ~ obese)) #P-value=0.2569
anova(lm(firm ~ bmi.class)) #P-value=0.1741
anova(lm(firm ~ bmi.class2)) #P-value=0.321

anova(lm(BFratio ~ obese)) #P-value=0.2508
anova(lm(BFratio ~ bmi.class)) #P-value=0.4966
anova(lm(BFratio ~ bmi.class2)) #P-value=0.506


###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(his.microb.edit ~ obese, permutations=1000) 
  #PERMANOVA=0.7153, pseudo-F=0.73249

set.seed(3)
adonis(his.microb.edit ~ bmi.class, permutations=1000) 
  #PERMANOVA=0.3357, pseudo-F=1.0772

set.seed(3)
adonis(his.microb.edit ~ bmi.class2, permutations=1000) 
  #PERMANOVA=0.3017, pseudo-F=1.1061


###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity
median(H) # 2.747097
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 56
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.6819666
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:31), 4])
table(test3[c(32:63), 4])
#Group1 (Higher than median), obese = 18 and non-obese = 13
#Group2 (Lower than median), obese = 20 and non-obese = 12

group1 <- c(18, 13)
group2 <- c(20, 12)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group1", "group2")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.08
## CI = 0.72, 1.61
## p-value = 0.719


##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 1.543046
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:31), 2]) #higher group
table(test4[c(32:63), 2]) #lower group
#Group1 (Higher than median), obese = 22 and non-obese = 9
#Group2 (Lower than median), obese = 16 and non-obese = 16

group1 <- c(22, 9)
group2 <- c(16, 16)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.70
## CI = 0.47, 1.07
## p-value = 0.089

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
edit.metadata2$obese.num[edit.metadata2$obese=="No"] <- 0
edit.metadata2$obese.num[edit.metadata2$obese=="Yes"] <- 1
obese <- factor(edit.metadata2$obese.num)

#generate test set
  # get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), his.microb.edit)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 3 Measures, AUCopt = 0.7526
plot(fit)
```
<br><br>

##### **Goodrich et al. 2014:**  

<br><br>

##### **Escobar et al. 2014:**  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

setwd("C:/users/marc/Desktop/obesity2/columbian/")

columbian.microb <- read.table("columbian.0.03.subsample.shared", header=T)
metadata <- read.csv("columbian_dataset.csv")

#Organize the microbiome shared data
rownames(columbian.microb) <- columbian.microb[, 2]
columbian.microb <- columbian.microb[, -c(1:3)]

#Organize the metadata
rownames(metadata) <- metadata[, 5]

#Get only data that we are interested in
edit.metadata <- metadata[, -c(1:7, 9, 13:15, 16:52)]

#Sort metadata into the same order as the microbiome data
order1 <- rownames(columbian.microb)
edit.metadata2 <- edit.metadata[order1, ]


#Get alpha diversity of the samples
library(vegan)

H <- diversity(columbian.microb)
S <- specnumber(columbian.microb)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(columbian.microb)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- columbian.microb
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)
#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Fusobacteria", "Lentisphaerae", "Spirochaetes", "Synergistetes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(4:5, 7:9)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
phyla.table.rel.abund <- phyla.table.rel.abund[, -8]


#Create Obese Yes/No groups
edit.metadata2$obese[edit.metadata2$description_s=="Adequate weight male stool sample" | edit.metadata2$description_s=="Overweight male stool sample" | edit.metadata2$description_s=="Adequate weight female stool sample" | edit.metadata2$description_s=="Overweight female stool sample"] <- "No"
edit.metadata2$obese[edit.metadata2$description_s=="Obese male stool sample" | edit.metadata2$description_s=="Obese female stool sample"] <- "Yes"

#Create BMI groups
edit.metadata2$BMI.class[edit.metadata2$body_mass_index_s<=24] <- "Normal"
edit.metadata2$BMI.class[edit.metadata2$body_mass_index_s>24 & edit.metadata2$body_mass_index_s<30] <- "Overweight"
edit.metadata2$BMI.class[edit.metadata2$body_mass_index_s>=30 & edit.metadata2$body_mass_index_s<40] <- "Obese"
edit.metadata2$BMI.class[edit.metadata2$body_mass_index_s>=40] <- "Extreme Obesity"

#Create BMI groups2
edit.metadata2$BMI.class2[edit.metadata2$body_mass_index_s<=24] <- "Normal"
edit.metadata2$BMI.class2[edit.metadata2$body_mass_index_s>24 & edit.metadata2$body_mass_index_s<30] <- "Overweight"
edit.metadata2$BMI.class2[edit.metadata2$body_mass_index_s>=30] <- "Obese"

#Get paitent demographics data to be tested 
bmi <- edit.metadata2$body_mass_index_s
obese <- factor(edit.metadata2$obese)
bmi.class <- factor(edit.metadata2$BMI.class)
bmi.class2 <- factor(edit.metadata2$BMI.class2)

######################################################################################## First Level Analysis & Alpha Diversity with BMI #############
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.8636
anova(lm(H ~ bmi.class)) #P-value=0.9584
anova(lm(H ~ bmi.class2)) #P-value=0.9584

anova(lm(S ~ obese)) #P-value=0.1947
anova(lm(S ~ bmi.class)) #P-value=0.1432
anova(lm(S ~ bmi.class2)) #P-value=0.1432

anova(lm(J ~ obese)) #P-value=0.4762
anova(lm(J ~ bmi.class)) #P-value=0.764
anova(lm(J ~ bmi.class2)) #P-value=0.764

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.7895, R2=0.002587
summary(lm(S ~ bmi)) #P-value=0.4474, R2=0.02076
summary(lm(J ~ bmi)) #P-value=0.9794, R2=2.423x10-5

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.09253
anova(lm(bacter ~ bmi.class)) #P-value=0.1742
anova(lm(bacter ~ bmi.class2)) #P-value=0.1742

anova(lm(firm ~ obese)) #P-value=0.3602
anova(lm(firm ~ bmi.class)) #P-value=0.5409
anova(lm(firm ~ bmi.class2)) #P-value=0.5409

anova(lm(BFratio ~ obese)) #P-value=0.4663
anova(lm(BFratio ~ bmi.class)) #P-value=0.7545
anova(lm(BFratio ~ bmi.class2)) #P-value=0.7545

###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################


set.seed(3)
adonis(columbian.microb ~ obese, permutations=1000) 
#PERMANOVA=0.08891, pseudo-F=1.3797

set.seed(3)
adonis(columbian.microb ~ bmi.class, permutations=1000) 
  #PERMANOVA=0.2358, pseudo-F=1.1142

set.seed(3)
adonis(columbian.microb ~ bmi.class2, permutations=1000) 
  #PERMANOVA=0.2358, pseudo-F=1.1142

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity
median(H) # 4.080262
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 362
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.6896595
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:15), 4])
table(test3[c(16:30), 4])
#Group1 (Higher than median), obese = 4 and non-obese = 11
#Group2 (Lower than median), obese = 6 and non-obese = 9

group1 <- c(4, 11)
group2 <- c(6, 9)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group1", "group2")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.50
## CI = 0.53, 4.26
## p-value = 0.439

##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.2516729
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:15), 2]) #higher group
table(test4[c(16:30), 2]) #lower group
#Group1 (Higher than median), obese = 7 and non-obese = 8
#Group2 (Lower than median), obese = 3 and non-obese = 12

group1 <- c(7, 8)
group2 <- c(3, 12)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

library(epiR)
epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.43
## CI = 0.14, 1.35
## p-value = 0.121

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
edit.metadata2$obese.num[edit.metadata2$obese=="No"] <- 0
edit.metadata2$obese.num[edit.metadata2$obese=="Yes"] <- 1
obese <- factor(edit.metadata2$obese.num)

#generate test set
  # get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), columbian.microb)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 3 Measures, AUCopt = 0.95
plot(fit)
```


##### **Zupancic et al. 2012:**  


```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################


setwd("C:/users/marc/Desktop/obesity2/Amish/updatedGOOD")

#Read in and match metadata to microbiome data
metadata <- read.csv("amish_obesity_table2.csv")
test <- metadata[!duplicated(metadata$submitted_sample_id_s), ]
rownames(test) <- test[, 4]
shared.data <- read.table("combined.0.03.subsample.shared", header=T)
rownames(shared.data) <- shared.data[, 2]
shared.data <- shared.data[, -c(1:3)]
keep  <- rownames(shared.data)

test2 <- test[keep, ]
good.metadata <- test2
rm(metadata, test, test2)

metadata <- good.metadata[!grepl("_s2", good.metadata$submitted_sample_id_s),]
keep <- rownames(metadata)
microbiome <- shared.data[keep, ]
rm(good.metadata, shared.data, keep)


#generate alpha diversity measures with vegan
library(vegan)
H <- diversity(microbiome)
S <- specnumber(microbiome)
J <- H/log(S)
select.alpha.diversity <- as.data.frame(cbind(H, S, J))
s1.alpha.diversity <- as.data.frame(select.alpha.diversity)
alpha.test <- s1.alpha.diversity

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(microbiome)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- microbiome
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)
#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Fusobacteria", "Spirochaetes")], 1, sum)
phyla.table <- phyla.table[, -c(4, 6)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:6)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100

#Create Obese Yes/No groups
# One problem is that we are assuming those with "not provided" are defaulting to the not affected group
metadata$obese[metadata$subject_is_affected_s=="<not provided>" | metadata$subject_is_affected_s=="No"] <- "No"
metadata$obese[metadata$subject_is_affected_s=="Yes"] <- "Yes"


#Create Obese.num group
metadata$obese.num[metadata$obese=="No"] <- 0
metadata$obese.num[metadata$obese=="Yes"] <- 1

#create groups to be used
obese <- factor(metadata$obese)


####################################################################################### First Level Analysis & Alpha Diversity with BMI #############
###########################################################################
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.2145
anova(lm(S ~ obese)) #P-value=0.05925
anova(lm(J ~ obese)) #P-value=0.5093

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.9485
anova(lm(firm ~ obese)) #P-value=0.2379
anova(lm(BFratio ~ obese)) #P-value=0.9277

####################################################################################### NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(microbiome ~ obese, permutations=1000) 
#PERMANOVA=0.5504, pseudo-F=0.97273

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity

median(H) # 3.073974
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 93
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.6790902
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:100), 4])
table(test3[c(101:201), 4])
#Group1 (Higher than median), obese = 29 and non-obese = 71
#Group2 (Lower than median), obese = 45 and non-obese = 56

group1 <- c(29, 71)
group2 <- c(45, 56)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.54
## CI = 1.05, 2.24
## p-value = 0.022


##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.537415
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:100), 2]) #higher group
table(test4[c(101:201), 2]) #lower group
#Group1 (Higher than median), obese = 38 and non-obese = 62
#Group2 (Lower than median), obese = 36 and non-obese = 65

group1 <- c(38, 62)
group2 <- c(36, 65)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.94
## CI = 0.65, 1.35
## p-value = 0.729

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
obese <- factor(metadata$obese.num)

#generate test set
# get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), microbiome)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 32 Measures, AUCopt = 0.6679613
plot(fit)
```

##### **Yatsunenko et al. 2012:**  


##### **Human Microbiome Project (HMP) 2011:**  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

##Read in relevant data

setwd("C:/users/marc/Desktop/obesity2/HMP.analysis")
microbiome <- read.table("Stool.an.0.03.subsample.shared", header=T)
meta.cat <- read.table("categorical.metadata", header=T)
meta.cont <- read.table("continuous.metadata", header=T)

#Only interested in first visit microbiome data
#need to create microbiome data set for only that

test <- microbiome[grep("\\.01\\.", microbiome$Group), ] #selecting by .01. 

microb.rownames <- gsub("([0-9]+).*", "\\1", test$Group) #extracting only numeric before first "."
rownames(test) <- microb.rownames

#Get rid of information not used in downstram analysis
test <- test[, -c(1:3)]

#Subset data to the microbiome total n
select.meta.cat <- meta.cat[microb.rownames, ]
select.meta.cont <- meta.cont[microb.rownames, ]

#generate alpha diversity measures with vegan
library(vegan)
H <- diversity(test)
S <- specnumber(test)
J <- H/log(S)
select.alpha.diversity <- as.data.frame(cbind(H, S, J))
alpha.test <- select.alpha.diversity

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(test)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- test
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)

#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Acidobacteria", "Deinococcus-Thermus", "Fusobacteria", "Lentisphaerae", "Spirochaetes", "Synergistetes", "Tenericutes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(1, 4, 6, 7, 9, 10:12)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
phyla.table.rel.abund <- phyla.table.rel.abund[, -8]


#Create Obese Yes/No groups
select.meta.cat$obese[select.meta.cat$BMI_C=="normal" | select.meta.cat$BMI_C=="overweight"] <- "No"
select.meta.cat$obese[select.meta.cat$BMI_C=="obese" | select.meta.cat$BMI_C=="extreme obesity"] <- "Yes"

#Create BMI groups2
select.meta.cat$BMI.class2[select.meta.cont$DTPBMI<=24] <- "Normal"
select.meta.cat$BMI.class2[select.meta.cont$DTPBMI>24 & select.meta.cont$DTPBMI<30] <- "Overweight"
select.meta.cat$BMI.class2[select.meta.cont$DTPBMI>=30] <- "Obese"

#Get paitent demographics data to be tested 
bmi <- select.meta.cont$DTPBMI
obese <- factor(select.meta.cat$obese)
bmi.class <- factor(select.meta.cat$BMI_C)
bmi.class2 <- factor(select.meta.cat$BMI.class2)

######################################################################################## First Level Analysis & Alpha Diversity with BMI #############
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.585
anova(lm(H ~ bmi.class)) #P-value=0.322
anova(lm(H ~ bmi.class2)) #P-value=0.322

anova(lm(S ~ obese)) #P-value=0.845
anova(lm(S ~ bmi.class)) #P-value=0.2051
anova(lm(S ~ bmi.class2)) #P-value=0.2051

anova(lm(J ~ obese)) #P-value=0.3559
anova(lm(J ~ bmi.class)) #P-value=0.3501
anova(lm(J ~ bmi.class2)) #P-value=0.3501

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.7041, R2=0.0005689
summary(lm(S ~ bmi)) #P-value=0.2304, R2=0.005657
summary(lm(J ~ bmi)) #P-value=0.962, R2=8.932x10-6

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.3518
anova(lm(bacter ~ bmi.class)) #P-value=0.1343
anova(lm(bacter ~ bmi.class2)) #P-value=0.1343

anova(lm(firm ~ obese)) #P-value=0.636
anova(lm(firm ~ bmi.class)) #P-value=0.07813
anova(lm(firm ~ bmi.class2)) #P-value=0.07813

anova(lm(BFratio ~ obese)) #P-value=0.5688
anova(lm(BFratio ~ bmi.class)) #P-value=0.6807
anova(lm(BFratio ~ bmi.class2)) #P-value=0.6807

###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(test ~ obese, permutations=1000) 
#PERMANOVA=0.8112, pseudo-F=0.7024

set.seed(3)
adonis(test ~ bmi.class, permutations=1000) 
  #PERMANOVA=0.8022, pseudo-F=0.77896

set.seed(3)
adonis(test ~ bmi.class2, permutations=1000) 
  #PERMANOVA=0.8022, pseudo-F=77896

######################################################################################## Relative Risk ###############################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity
median(H) # 2.541824
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 57
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.6305742
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:128), 4])
table(test3[c(129:256), 4])
#Group1 (Higher than median), obese = 10 and non-obese = 118
#Group2 (Lower than median), obese = 16 and non-obese = 112

group1 <- c(10, 118)
group2 <- c(16, 112)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group1", "group2")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.60
## CI = 0.75, 3.39
## p-value = 0.214

##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 2.529168
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:128), 2]) #higher group
table(test4[c(129:256), 2]) #lower group
#Group1 (Higher than median), obese = 13 and non-obese = 115
#Group2 (Lower than median), obese = 13 and non-obese = 115

group1 <- c(13, 115)
group2 <- c(13, 115)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

library(epiR)
epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.00
## CI = 0.48, 2.07
## p-value = 1.00

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
select.meta.cat$obese.num[select.meta.cat$obese=="No"] <- 0
select.meta.cat$obese.num[select.meta.cat$obese=="Yes"] <- 1
obese <- factor(select.meta.cat$obese.num)

#generate test set
  # get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), test)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 6 Measures, AUCopt = 0.7031773
plot(fit)
```

##### **Nam, et al. 2011:**  
<br>
It should be noted that for the grouping part of this analysis it was done in relationship to being overweight instead of obese. But for the combined portion of the analysis or third arm (all data sets taken together) it was used.  Conclusions on the other arms are not dependent on this specific data set. 

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

setwd("C:/users/marc/Desktop/obesity2/korean/")

microb <- read.table("DRR000776.0.03.subsample.shared", header=T)
rownames(microb) <- microb[, 2]
microb <- microb[,-c(1:3)]

metadata <- read.csv("korean.metadata.csv")

#Need to get rid of the repeated samples and the children
  #First the metadata
metadata2 <- metadata[-c(2:3, 5:6, 8:9, 11:12, 14:15, 17:18, 19:24), ]
rownames(metadata2) <- metadata2[, 2]
metadata2 <- metadata2[, -c(1:2)]

  #Second the microbiome data set
microb2 <- microb[-c(2:3, 5:6, 8:9, 11:12, 14:15, 17:18, 19:24), ]
rownames(microb2) <- rownames(metadata2)

#Get alpha diversity of the samples
library(vegan)

H <- diversity(microb2)
S <- specnumber(microb2)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)
# Seems to look okay....

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(microb2)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- microb2
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)
#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Fusobacteria", "Lentisphaerae", "Synergistetes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(4:5, 7:8)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
phyla.table.rel.abund <- phyla.table.rel.abund[, -8]


#Create BMI groups
metadata2$BMI.class[metadata2$BMI<=24] <- "Normal"
metadata2$BMI.class[metadata2$BMI>24 & metadata2$BMI<30] <- "Overweight"
metadata2$BMI.class[metadata2$BMI>=30 & metadata2$BMI<40] <- "Obese"
metadata2$BMI.class[metadata2$BMI>=40] <- "Extreme Obesity"

#Create Overweight Yes/No groups
metadata2$overweight[metadata2$BMI.class=="Normal"] <- "No"
metadata2$overweight[metadata2$BMI.class=="Obese" | metadata2$BMI.class=="Extreme Obesity" | metadata2$BMI.class=="Overweight"] <- "Yes"

#Get paitent demographics data to be tested 
bmi <- metadata2$BMI
overweight <- factor(metadata2$overweight)


######################################################################################## First Level Analysis & Alpha Diversity with BMI #############
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ overweight)) #P-value=0.961
anova(lm(S ~ overweight)) #P-value=0.5165
anova(lm(J ~ overweight)) #P-value=0.6547

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.8838, R2=0.001376
summary(lm(S ~ bmi)) #P-value=0.3153, R2=0.06293
summary(lm(J ~ bmi)) #P-value=0.7022, R2=0.009386

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ overweight)) #P-value=0.4382
anova(lm(firm ~ overweight)) #P-value=0.2609
anova(lm(BFratio ~ overweight)) #P-value=0.4089

###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(microb2 ~ overweight, permutations=1000) 
#PERMANOVA=0.4236, pseudo-F=0.97425

######################################################################################## Relative Risk ###############################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity
median(H) # 3.888484
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 156.5
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.7715805
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(overweight)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:9), 4])
table(test3[c(10:18), 4])
#Group1 (Higher than median), overweight = 4 and normal = 5
#Group2 (Lower than median), overweight = 2 and normal = 7

group1 <- c(4, 5)
group2 <- c(2, 7)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Overweight", "normal")
rownames(r.test) <- c("group1", "group2")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.50
## CI = 0.12, 2.08
## p-value = 0.317

##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.4243408
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, overweight)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:9), 2]) #higher group
table(test4[c(10:18), 2]) #lower group
#Group1 (Higher than median), overweight = 2 and normal = 7
#Group2 (Lower than median), overweight = 4 and normal = 5

group1 <- c(2, 7)
group2 <- c(4, 5)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

library(epiR)
epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 2.00
## CI = 0.48, 8.31
## p-value = 0.317


###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
metadata2$over.num[metadata2$overweight=="No"] <- 0
metadata2$over.num[metadata2$overweight=="Yes"] <- 1
overweight <- factor(metadata2$over.num)

#generate test set
  # get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), microb2)
testset <- cbind(overweight, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=500, nodesize=1)
  #Since this data set was so small had to modify the standardized        
  #parameters.
summary(fit) # list of 2 Measures, AUCopt = 1.00
plot(fit)
```

##### **Wu, et al. 2011:**  


```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################


setwd("C:/users/marc/Desktop/obesity2/COMBO/")

microbiome <- read.table("combined.good.subsample.shared", header=T)
rownames(microbiome) <- microbiome$Group
microbiome <- microbiome[, -c(1:3)]
seqData <- read.csv("COMBO_data_table.csv", header=T)
seqData <- seqData[-1, ]
rownames(seqData) <- seqData$Run_s
metadata <- read.table("BMI_and_Counts.txt", header=T)
metadata <- metadata[, c(1, 2)]

#get seqData set in line with microbiome data
namesToKeep <- rownames(microbiome)
test <- seqData[namesToKeep, ]
seqData <- test
rm(test)

#change all rownames to match sample IDs
rownames(seqData) <- seqData$submitted_subject_id_s
rownames(microbiome) <- rownames(seqData)
rownames(metadata) <- metadata[, 1]

#Match the metadata now with the microbiome data
namesToKeep <- rownames(microbiome)
test <- metadata[namesToKeep, ]
metadata <- test
rm(test)
rm(seqData, keep, namesToKeep)

#Get alpha diversity of the samples
library(vegan)

H <- diversity(microbiome)
S <- specnumber(microbiome)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.table("combined.good.phyla.txt", header=T)
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(microbiome)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- microbiome
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)
#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Fusobacteria", "Synergistetes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(4, 6:7)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:6)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
rm(keep, phyla.names, phyla.total, phylogenetic.info, phyla.table, phyla.good)

#Create BMI groups
metadata$BMI.class[metadata$BMI<=24] <- "Normal"
metadata$BMI.class[metadata$BMI>24 & metadata$BMI<30] <- "Overweight"
metadata$BMI.class[metadata$BMI>=30 & metadata$BMI<40] <- "Obese"
metadata$BMI.class[metadata$BMI>=40] <- "Extreme Obesity"

#Create Obese Yes/No groups
metadata$obese[metadata$BMI.class=="Normal" | metadata$BMI.class=="Overweight"] <- "No"
metadata$obese[metadata$BMI.class=="Obese" | metadata$BMI.class=="Extreme Obesity"] <- "Yes"

#Create a column with obese and extreme obese as single entity
metadata$BMIclass2[metadata$BMI.class=="Normal"] <- "Normal"
metadata$BMIclass2[metadata$BMI.class=="Overweight"] <- "Overweight"                     
metadata$BMIclass2[metadata$BMI.class=="Obese" | metadata$BMI.class=="Extreme Obesity"] <- "Obese"

#Get paitent demographics to be tested
bmi <- metadata$BMI
obese <- factor(metadata$obese)
bmi.class <- factor(metadata$BMI.class)
bmi.class2 <- factor(metadata$BMIclass2)


###########################################################################
######### First Level Analysis & Alpha Diversity with BMI #################
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.7493
anova(lm(H ~ bmi.class)) #P-value=0.2959
anova(lm(H ~ bmi.class2)) #P-value=0.2089

anova(lm(S ~ obese)) #P-value=0.7511
anova(lm(S ~ bmi.class)) #P-value=0.5128
anova(lm(S ~ bmi.class2)) #P-value=0.3891

anova(lm(J ~ obese)) #P-value=0.5557
anova(lm(J ~ bmi.class)) #P-value=0.2647
anova(lm(J ~ bmi.class2)) #P-value=0.1818

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.8276, R2=0.0008541
summary(lm(S ~ bmi)) #P-value=0.8195, R2=0.0009377
summary(lm(J ~ bmi)) #P-value=0.8521, R2=0.0006262

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.9173
anova(lm(bacter ~ bmi.class)) #P-value=0.5554
anova(lm(bacter ~ bmi.class2)) #P-value=0.6846

anova(lm(firm ~ obese)) #P-value=0.5765
anova(lm(firm ~ bmi.class)) #P-value=0.2157
anova(lm(firm ~ bmi.class2)) #P-value=0.2817

anova(lm(BFratio ~ obese)) #P-value=0.6735
anova(lm(BFratio ~ bmi.class)) #P-value=0.6208
anova(lm(BFratio ~ bmi.class2)) #P-value=0.4681

###########################################################################
############ NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(microbiome ~ obese, permutations=1000) 
#PERMANOVA=0.8921, pseudo-F=0.66065

set.seed(3)
adonis(microbiome ~ bmi.class, permutations=1000) 
#PERMANOVA=0.9321, pseudo-F=0.73921

set.seed(3)
adonis(microbiome ~ bmi.class2, permutations=1000) 
#PERMANOVA=0.75922, pseudo-F=0.8791

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity
median(H) # 3.000655
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 98.5
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.6432727
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:29), 4])
table(test3[c(30:58), 4])
#Group1 (Higher than median), obese = 3 and non-obese = 26
#Group2 (Lower than median), obese = 2 and non-obese = 27

group1 <- c(3, 26)
group2 <- c(2, 27)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.67
## CI = 0.12, 3.70
## p-value = 0.64

##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 2.438335
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:29), 2]) #higher group
table(test4[c(30:58), 2]) #lower group
#Group1 (Higher than median), obese = 3 and non-obese = 26
#Group2 (Lower than median), obese = 2 and non-obese = 27

group1 <- c(3, 26)
group2 <- c(2, 27)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.67
## CI = 0.12, 3.70
## p-value = 0.64

###########################################################################
############ Classification using AUCRF####################################
###########################################################################

library(AUCRF)

#Create Obese.num group
metadata$obese.num[metadata$obese=="No"] <- 0
metadata$obese.num[metadata$obese=="Yes"] <- 1
obese <- factor(metadata$obese.num)

#generate test set
# get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), microbiome)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 15 Measures, AUCopt = 0.7075472
plot(fit)
```

##### **Arumugam, et al. 2010 (MetaHit):**  
<br>
This first R analysis chunk looks at how similar metaphlan2 is with 16S sequencing in the HMP data set.

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

#Phyla first then will explore other measures if data is promising

setwd("C:/Users/marc/Desktop/obesity2/HMP.analysis")

metaphlanData <- read.table("HMP_metaphlan2_merged_abundance_table.txt", header=T)
IDconversion <- read.csv("HMP.stool.IDs.csv")
microbiome <- read.table("Stool.an.0.03.subsample.shared", header=T)

#Only interested in first visit microbiome data
#need to create microbiome data set for only that

Data16S <- microbiome[grep("\\.01\\.", microbiome$Group), ] #selecting by .01. 

microb.rownames <- gsub("([0-9]+).*", "\\1", Data16S$Group) #extracting only numeric before first "."
rownames(Data16S) <- microb.rownames

#Get rid of information not used in downstram analysis
Data16S <- Data16S[, -c(1:3)]

#Get phyla information
#Edited out non phyla information first with sed in linux
#combined new labels with previous taxonomy file with excel
phylogenetic.info <- read.csv("phyla.data.csv")
rownames(phylogenetic.info) <- phylogenetic.info[,1]
phylogenetic.info <- phylogenetic.info[,-c(1)]
phyla.names <- as.character(phylogenetic.info$Taxonomy)
keep <- colnames(Data16S)
phyla.good <- phylogenetic.info[keep, ]
phyla.names <- as.character(phyla.good[,2])
phyla.table <- Data16S
colnames(phyla.table) <- phyla.names
#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing, keep, microb.rownames, phyla.names)

#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Acidobacteria", "Deinococcus-Thermus", "Fusobacteria", "Lentisphaerae", "Spirochaetes", "Synergistetes", "Tenericutes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(1, 4, 6, 7, 9, 10:12)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
rm(phyla.total)

# Create comparable data set with 
metaphlanPhlyaData <- read.csv("HMP.metaphlan.phyla.data.csv", header=T)
rownames(metaphlanPhlyaData) <- metaphlanPhlyaData$ID
metaphlanPhlyaData <- metaphlanPhlyaData[,-1]
metaphlanPhlyaData <- as.data.frame(t(metaphlanPhlyaData))

#combine phyla that are not that abundant
metaphlanPhlyaData$other <- apply(metaphlanPhlyaData[, c("Acidobacteria", "Candidatus_Saccharibacteria", "Deinococcus_Thermus", "Fusobacteria", "Spirochaetes", "Synergistetes", "Tenericutes")], 1, sum)
metaphlanPhlyaData <- metaphlanPhlyaData[, -c(1, 4:5, 7, 9:11)]

#Create a relative abundance table for phyla
metaphlanPhlyaData.total <- apply(metaphlanPhlyaData[, c(1:6)], 1, sum)
metaphlanPhlyaData.rel.abund <- (metaphlanPhlyaData/metaphlanPhlyaData.total)*100

#Need to get sampleIDs to match 16S data
metaphlanNames <- rownames(metaphlanPhlyaData)
rownames(IDconversion) <- IDconversion[,1]
matchedData <- IDconversion[metaphlanNames, c(1,3)]
IndIDmatched <- matchedData[,2]

#Need to find and remove duplicated values (No idea which sample goes with which)
test1 <- as.data.frame(table(matchedData$Individual.ID))
good <- which(test1$Freq == 1)
unique <- test1[good, ]
uniqueNames <- as.character(unique$Var1)

test <- phyla.table.rel.abund[rownames(phyla.table.rel.abund) %in% uniqueNames, ]
finalUniqueNames <- rownames(test)
UpdatedMatchedData <- matchedData[matchedData$Individual.ID %in% finalUniqueNames, ]

goodMetaphlanPhylaData.rel.abund <- metaphlanPhlyaData.rel.abund[rownames(metaphlanPhlyaData.rel.abund) %in% rownames(UpdatedMatchedData), ]
rownames(goodMetaphlanPhylaData.rel.abund) <- UpdatedMatchedData[, 2]
testing <- goodMetaphlanPhylaData.rel.abund[order(rownames(goodMetaphlanPhylaData.rel.abund)), ]

phyla16SData <- test
phylaMetaphlanData <- testing
rm(finalUniqueNames, good, IndIDmatched, metaphlanNames, metaphlanPhlyaData.total, uniqueNames, goodMetaphlanPhylaData.rel.abund, matchedData, metaphlanPhlyaData, metaphlanPhlyaData.rel.abund, phyla.good, phyla.table, phyla.table.rel.abund, phylogenetic.info, test, test1, testing, unique, UpdatedMatchedData)

###########################################################################
############ Let's Get to the Actual Comparison ###########################
###########################################################################

# Can start off with a general bar plot of the two side by side 
library(qdap)
library(plyr)
library(reshape2)
library(ggplot2)
library(RColorBrewer)
library(scales)
library(gridExtra) #using grid.arrange() can add different plots together
library(plotrix)

#phyla16SData <- phyla16SData[,-5] only if want to ignore unclassified
phylaMetaphlanData$unclassified <- 0
phylaMetaphlanData <- subset(phylaMetaphlanData, select=c(Actinobacteria:Proteobacteria, unclassified, Verrucomicrobia, other))
phyla16SData$dataset <- 1 
phylaMetaphlanData$dataset <- 2

testset <- rbind(phyla16SData, phylaMetaphlanData)
testset$dataset <- factor(testset$dataset)
melt.testset <- melt(testset, id.vars=c("dataset"))
colnames(melt.testset)[2] <- "phyla"
barplotAnalysisData <- ddply(melt.testset, c("dataset", "phyla"), summarise, mean = mean(value), sd=sd(value), sem=sd(value/sqrt(length(value))))
  
bpg1 <- ggplot(barplotAnalysisData, aes(x=dataset, y=mean, fill=phyla)) + geom_bar(position = "fill", stat="identity")
bpg2 <- bpg1 + scale_fill_brewer(palette="Set1") + scale_y_continuous(label=percent)
bpg2

# Now can look at direct linear correlation between same sample
# Between metaphlan2 and 16S amplicon sequencing

#Actinobacteria
summary(lm(phyla16SData$Actinobacteria ~ phylaMetaphlanData$Actinobacteria)) 
#P-value 0.5543, R2= 0.006899
plot(phyla16SData$Actinobacteria ~ phylaMetaphlanData$Actinobacteria)
abline(lm(phyla16SData$Actinobacteria ~ phylaMetaphlanData$Actinobacteria), col="red")

#Bacteroidetes
summary(lm(phyla16SData$Bacteroidetes ~ phylaMetaphlanData$Bacteroidetes)) 
#P-value 0.0002277, R2= 0.2358
plot(phyla16SData$Bacteroidetes ~ phylaMetaphlanData$Bacteroidetes)
abline(lm(phyla16SData$Bacteroidetes ~ phylaMetaphlanData$Bacteroidetes), col="red")

#Firmicutes
summary(lm(phyla16SData$Firmicutes ~ phylaMetaphlanData$Firmicutes)) 
#P-value 0.1223, R2= 0.04617
plot(phyla16SData$Firmicutes ~ phylaMetaphlanData$Firmicutes)
abline(lm(phyla16SData$Firmicutes ~ phylaMetaphlanData$Firmicutes), col="red")

#Proteobacteria
summary(lm(phyla16SData$Proteobacteria ~ phylaMetaphlanData$Proteobacteria)) 
#P-value 0.004252, R2= 0.1494
plot(phyla16SData$Proteobacteria ~ phylaMetaphlanData$Proteobacteria)
abline(lm(phyla16SData$Proteobacteria ~ phylaMetaphlanData$Proteobacteria), col="red")

#Verrucomicrobia
summary(lm(phyla16SData$Verrucomicrobia ~ phylaMetaphlanData$Verrucomicrobia)) 
#P-value 0.1761, R2= 0.03559
plot(phyla16SData$Verrucomicrobia ~ phylaMetaphlanData$Verrucomicrobia)
abline(lm(phyla16SData$Verrucomicrobia ~ phylaMetaphlanData$Verrucomicrobia), col="red")

#other
summary(lm(phyla16SData$other ~ phylaMetaphlanData$other)) 
#P-value 0.8054, R2= 0.001201
plot(phyla16SData$other ~ phylaMetaphlanData$other)
abline(lm(phyla16SData$other ~ phylaMetaphlanData$other), col="red")

# Create a bland Altman of the best correlation
  # bias (difference of metaphlan) based on 16S analysis  
library(BlandAltmanLeh)
bland.altman.plot(phyla16SData$Bacteroidetes, phylaMetaphlanData$Bacteroidetes, main="Bland Altman Plot of Bacteroidetes", xlab="Means", ylab="Differences")
```


<br>
This second R analysis chunk uses metaphlan2 to look at the questions we were tackling with the other data sets indivdiually.  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

setwd("C:/users/marc/Desktop/obesity2/MetaHit/")

microb <- read.csv("finalized_metaHit_shared.csv")
rownames(microb) <- microb[, 1]
microb <- microb[,-1]

metadata <- read.csv("MetaHit_metadata.csv")
rownames(metadata) <- metadata[, 1]
metadata <- metadata[, -1]

phyla <- read.csv("phyla.csv")
rownames(phyla) <- phyla[, 1]
phyla <- phyla[, -1]

# Generate Overall Relative Abundance - normalize to a value of 100
# Needs to be done since not all measures have 100 bacteria from metaphlan2
overall <- rowSums(microb)
microb.norm <- (microb / overall) * 100

phyla.table <- read.csv("phyla.csv", header = T)
rownames(phyla.table) <- phyla.table[, 1]
phyla.table <- phyla.table[, -1]
phyla.table <- as.data.frame(t(phyla.table))

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:10)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
rm(phyla.table, phyla.info)

#Get alpha diversity of the samples
library(vegan)

H <- diversity(microb.norm)
S <- specnumber(microb.norm)
J <- H/log(S)
alpha.diversity.shannon <- cbind(H,S,J)
alpha.test <- as.data.frame(alpha.diversity.shannon)
# Seems to look okay....

#Create BMI groups
metadata$BMI.class[metadata$BMI<=24] <- "Normal"
metadata$BMI.class[metadata$BMI>24 & metadata$BMI<30] <- "Overweight"
metadata$BMI.class[metadata$BMI>=30 & metadata$BMI<40] <- "Obese"
metadata$BMI.class[metadata$BMI>=40] <- "Extreme Obesity"

#Create Obese Yes/No groups
metadata$obese[metadata$BMI.class=="Normal" | metadata$BMI.class=="Overweight"] <- "No"
metadata$obese[metadata$BMI.class=="Obese" | metadata$BMI.class=="Extreme Obesity"] <- "Yes"


#Create Obese.num group
metadata$obese.num[metadata$obese=="No"] <- 0
metadata$obese.num[metadata$obese=="Yes"] <- 1


#Create a column with obese and extreme obese as single entity
metadata$BMIclass2[metadata$BMI<=24] <- "Normal"
metadata$BMIclass2[metadata$BMI>24 & metadata$BMI<30] <- "Overweight"      
metadata$BMIclass2[metadata$BMI>=30] <- "Obese"


#Get paitent demographics to be tested
bmi <- metadata$BMI
obese <- factor(metadata$obese)
bmi.class <- factor(metadata$BMI.class)
bmi.class2 <- factor(metadata$BMIclass2)

####################################################################################### First Level Analysis & Alpha Diversity with BMI #############
###########################################################################
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.2174
anova(lm(H ~ bmi.class)) #P-value=0.5339
anova(lm(H ~ bmi.class2)) #P-value=0.3628

anova(lm(S ~ obese)) #P-value=0.9878
anova(lm(S ~ bmi.class)) #P-value=0.7083
anova(lm(S ~ bmi.class2)) #P-value=0.4985

anova(lm(J ~ obese)) #P-value=0.1874
anova(lm(J ~ bmi.class)) #P-value=0.3772
anova(lm(J ~ bmi.class2)) #P-value=0.2326

##linear correlations
summary(lm(H ~ bmi)) #P-value=0.1022, R2=0.03185
summary(lm(S ~ bmi)) #P-value=0.7642, R2=0.001091
summary(lm(J ~ bmi)) #P-value=0.06682, R2=0.0399

#B and F tests against obesity
bacter <- phyla.table.rel.abund$Bacteroidetes
firm <- phyla.table.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.1911
anova(lm(bacter ~ bmi.class)) #P-value=0.2975
anova(lm(bacter ~ bmi.class2)) #P-value=0.2263

anova(lm(firm ~ obese)) #P-value=0.2925
anova(lm(firm ~ bmi.class)) #P-value=0.3654
anova(lm(firm ~ bmi.class2)) #P-value=0.2998

anova(lm(BFratio ~ obese)) #P-value=0.05254
anova(lm(BFratio ~ bmi.class)) #P-value=0.1808
anova(lm(BFratio ~ bmi.class2)) #P-value=0.1284

####################################################################################### NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(microb.norm ~ obese, permutations=1000) 
#PERMANOVA=0.05495, pseudo-F=1.6047

set.seed(3)
adonis(microb.norm ~ bmi.class, permutations=1000) 
#PERMANOVA=0.1798, pseudo-F=1.176

set.seed(3)
adonis(microb.norm ~ bmi.class2, permutations=1000) 
#PERMANOVA=0.2138, pseudo-F=1.1686

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity

median(H) # 2.886851
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 85
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.664365
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:42), 4])
table(test3[c(43:85), 4])
#Group1 (Higher than median), obese = 17 and non-obese = 25
#Group2 (Lower than median), obese = 20 and non-obese = 23

group1 <- c(17, 25)
group2 <- c(20, 23)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.15
## CI = 0.71, 1.87
## p-value = 0.575


##Run the RR for B/F ratio
Bacter = phyla.table.rel.abund$Bacteroidetes
Firm = phyla.table.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.9224021
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:42), 2]) #higher group
table(test4[c(43:85), 2]) #lower group
#Group1 (Higher than median), obese = 20 and non-obese = 22
#Group2 (Lower than median), obese = 17 and non-obese = 26

group1 <- c(20, 22)
group2 <- c(17, 26)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.83
## CI = 0.51, 1.35
## p-value = 0.452

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
metadata$obese.num[metadata$obese=="No"] <- 0
metadata$obese.num[metadata$obese=="Yes"] <- 1
obese <- factor(metadata$obese.num)

#generate test set
# get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), microb.norm)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, phyla.table.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 19 Measures, AUCopt = 0.7649212
plot(fit)
```


##### **Turnbaugh, et al. 2008:**  

```r
###########################################################################
############ Preparing Data Tables for Analysis ###########################
###########################################################################

setwd("C:/users/marc/Desktop/obesity2/turnbaugh.twins")
shared.data <- read.table("test.unique.good.filter.unique.precluster.pick.pick.an.shared", header=T)
rownames(shared.data) <- shared.data[, 2]
shared.data <- shared.data[, -c(1:3)]

subsample.data <- read.table("test.unique.good.filter.unique.precluster.pick.pick.an.0.03.subsample.shared", header=T)
rownames(subsample.data) <- subsample.data[, 2]
subsample.data <- subsample.data[, -c(1:3)]


metadata <- read.csv("turnbaugh.metadata.csv")
rownames(metadata) <- metadata[, 4]
metadata <- metadata[, -4]

keep1 <- which(metadata$Sample == 1) # n =146

#use the first sampling
#subset data for only the first sampling
s1.metadata <- metadata[keep1, ]
s1.subsample.data <- subsample.data[keep1, ]


#generate alpha diversity measures with vegan
library(vegan)
H <- diversity(s1.subsample.data)
S <- specnumber(s1.subsample.data)
J <- H/log(S)
select.alpha.diversity <- as.data.frame(cbind(H, S, J))
s1.alpha.diversity <- as.data.frame(select.alpha.diversity)
alpha.test <- s1.alpha.diversity

#Generate phyla table data

phyla.info <- read.csv("phyla.csv")
phyla.table <- shared.data
colnames(phyla.table) <- phyla.info[, 2]


#add all the same columns up and then return the sum
testing <- t(rowsum(t(phyla.table), group = rownames(t(phyla.table))))
phyla.table <- as.data.frame(testing)
rm(testing)

#combine phyla that are not that abundant
phyla.table$other <- apply(phyla.table[, c("Fusobacteria", "Lentisphaerae", "Spirochaetes", "Synergistetes", "TM7")], 1, sum)
phyla.table <- phyla.table[, -c(4:5, 7:9)]

#Create a relative abundance table for phyla
phyla.total <- apply(phyla.table[, c(1:7)], 1, sum)
phyla.table.rel.abund <- (phyla.table/phyla.total)*100
s1.phyla.rel.abund <- phyla.table.rel.abund[rownames(shared.data), ]
s1.phyla.rel.abund <- s1.phyla.rel.abund[keep1, ]

#Create Obese Yes/No groups
s1.metadata$obese[s1.metadata$BMI.category=="Lean" | s1.metadata$BMI.category=="Overweight"] <- "No"
s1.metadata$obese[s1.metadata$BMI.category=="Obese"] <- "Yes"

#Create Obese.num group
s1.metadata$obese.num[s1.metadata$obese=="No"] <- 0
s1.metadata$obese.num[s1.metadata$obese=="Yes"] <- 1

#create groups to be used
obese <- factor(s1.metadata$obese)
bmi <- s1.metadata$BMI.category

####################################################################################### First Level Analysis & Alpha Diversity with BMI #############
###########################################################################
###########################################################################

##Test BMI versus alpha diversity and phyla

anova(lm(H ~ obese)) #P-value=0.06976
anova(lm(S ~ obese)) #P-value=0.02435
anova(lm(J ~ obese)) #P-value=0.1253


##linear correlations
summary(lm(H ~ bmi)) #P-value=0.1926, R2=0.02277
summary(lm(S ~ bmi)) #P-value=0.03434, R2=0.04606
summary(lm(J ~ bmi)) #P-value=0.2988, R2=0.01675

#B and F tests against obesity
bacter <- s1.phyla.rel.abund$Bacteroidetes
firm <- s1.phyla.rel.abund$Firmicutes
BFratio <- bacter/firm

anova(lm(bacter ~ obese)) #P-value=0.5771
anova(lm(firm ~ obese)) #P-value=0.4073
anova(lm(BFratio ~ obese)) #P-value=0.2216

####################################################################################### NMDS and PERMANOVA Analysis###################################
###########################################################################

set.seed(3)
adonis(s1.subsample.data ~ obese, permutations=1000) 
#PERMANOVA=0.09491, pseudo-F=1.2114

###########################################################################
############ Relative Risk#################################################
###########################################################################

library(epiR)

#Generate median values and put them into existing alpha.test dataframe
#Shannon diversity

median(H) # 4.327308
alpha.test <- within(alpha.test, {shannon.cat = ifelse(H <= median(H), "less", "higher")})

#OTU Richness
median(S) # 290.5
alpha.test <- within(alpha.test, {S.cat = ifelse(S <= median(S), "less", "higher")})

#Evenness
median(J) # 0.7611711
alpha.test <- within(alpha.test, {J.cat = ifelse(J <= median(J), "less", "higher")})

##Shannon Diversity
H.cat <- alpha.test$shannon.cat
S.cat <- alpha.test$S.cat
J.cat <- alpha.test$J.cat
bmi.cat <- as.character(obese)
test3 <- cbind(H.cat, S.cat, J.cat, bmi.cat)
test3 <- test3[order(H.cat), ]
table(test3[c(1:73), 4])
table(test3[c(74:146), 4])
#Group1 (Higher than median), obese = 47 and non-obese = 26
#Group2 (Lower than median), obese = 52 and non-obese = 21

group1 <- c(47, 26)
group2 <- c(52, 21)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 1.11
## CI = 0.88, 1.38
## p-value = 0.376


##Run the RR for B/F ratio
Bacter = s1.phyla.rel.abund$Bacteroidetes
Firm = s1.phyla.rel.abund$Firmicutes
BFRatio = Bacter/Firm
BFRatio <- as.data.frame(BFRatio)

median(BFRatio$BFRatio) # 0.4764057
BFRatio <- within(BFRatio, {BFRatio.cat = ifelse(BFRatio <= median(BFRatio), "less", "higher")})

BFRatio.cat <- BFRatio$BFRatio.cat
test4 <- cbind(BFRatio.cat, obese)
test4 <- test4[order(BFRatio.cat), ]
table(test4[c(1:73), 2]) #higher group
table(test4[c(74:146), 2]) #lower group
#Group1 (Higher than median), obese = 54 and non-obese = 19
#Group2 (Lower than median), obese = 45 and non-obese = 28

group1 <- c(54, 19)
group2 <- c(45, 28)
r.test <- rbind(group2, group1)
colnames(r.test) <- c("Obese", "Not.Obese")
rownames(r.test) <- c("group2", "group1")

epi.2by2(r.test, method="cohort.count")
## Risk Ratio = 0.83
## CI = 0.66, 1.05
## p-value = 0.111

###########################################################################
############ Classification using AUCRF ###################################
###########################################################################

library(AUCRF)

#Create Obese.num group
s1.metadata$obese.num[s1.metadata$obese=="No"] <- 0
s1.metadata$obese.num[s1.metadata$obese=="Yes"] <- 1
obese <- factor(s1.metadata$obese.num)

#generate test set
# get rid of those with 0 and only 4 other values
testset <- Filter(function(x)(length(unique(x))>5), s1.subsample.data)
testset <- cbind(obese, testset)
colnames(testset)[1] <- "obese"
testset <- cbind(testset, H, S, J, s1.phyla.rel.abund)

#Try AUCRF with default measures provided in readme
set.seed(3)
fit <- AUCRF(obese ~ ., data=testset, ntree=1000, nodesize=20)
summary(fit) # list of 11 Measures, AUCopt = 0.7764883
plot(fit)
```


#### **Part 2: Pooled Meta-analysis RR of Obesity for Shannon Diversity and B/F Ratio**    
<br>
This first R chunk shows the code used to generate the RR for obesity and Shannon diversity.  

```r
#Load required packages
library(meta)
library(metafor) #This package is used mostly for the more in-depth graph
library(rmeta)  #This package was used for the less intensive graph

#Change working directory
setwd("C:/users/marc/Desktop/obesity2")

StudyNo <- c(1, 2, 3, 4, 5, 6, 7, 8)
Study <- c("Baxter", "HMP", "Ross", "Escobar", "Argumugam", "Turnbaugh", "Zupancic", "Wu")
Year <- c("2015", "2011", "2015", "2014", "2010", "2009", "2012", "2012")
Total <- c("n=172", "n=256", "n=63", "n=30", "n=85", "n=146", "n=201", "n=58")
tpos <- c(25, 16, 20, 6, 20, 52, 45, 2) #obese in low diversity group
tneg <- c(61, 112, 12, 9, 23, 21, 56, 27) #nonobese in low diversity group
cpos <- c(22, 10, 18, 4, 17, 47, 29, 3) #obese in high diversity group
cneg <- c(64, 118, 13, 11, 25, 26, 71, 26) #nonobese in high diversity group
RR <- c(1.14, 1.6, 1.08, 1.50, 1.15, 1.11, 1.54, 0.67)
low <- c(0.70, 0.75, 0.72, 0.53, 0.71, 0.88, 1.05, 0.12)
high <- c(1.85, 3.39, 1.61, 4.26, 1.87, 1.38, 2.24, 3.70)

numeric.data <-as.data.frame(cbind(tpos, tneg, cpos, cneg))
raw.overall <- as.data.frame(cbind(StudyNo, Study,Year, Total, numeric.data))

dat <- escalc(measure = "RR", ai = tpos, bi = tneg, ci = cpos, di = cneg, data=raw.overall, append=TRUE)
k <- length(dat$StudyNo)
dat.fm <- data.frame(study=factor(rep(1:k, each = 4)))
dat.fm$grp <- factor(rep(c("T", "T", "C", "C"), k), levels = c("T", "C"))
dat.fm$out <- factor(rep(c("+", "-", "+", "-"), k), levels = c("+", "-"))
dat.fm$freq <- with(raw.overall, c(rbind(tpos, tneg, cpos, cneg)))

escalc(out ~ grp | study, weights = freq, data = dat.fm, measure = "RR")


#Used to generate the RE Model estimate based on all the studies
res <- rma(ai=tpos, bi=tneg, ci=cpos, di=cneg, data=raw.overall, measure="RR", slab=paste(Study, Year, Total, sep=", "), method="REML")


#To create a graph visualize the data
forest(res, at=log(c(0.3, 1, 5)), atransf=exp, xlab="RR of Obesity in high versus low Shannon diversity based on Median", cex=0.8)
par(font=2) #Switch to bold

# RR Title ################################################
#text(3.2, 3.10, "Relative Risk [95% CI]", pos=2, cex=0.8)#
  #large scale (Plot in R studio at full)                 #
                                                          #
text(5.5, 9.3, "Relative Risk [95% CI]", pos=2, cex=0.8)  #
  #smaller scale (Plot in R studia at normal)             #
###########################################################

# Group Title#############################################
                                                         #
text(-6.5, 9.3, "Study, Year, and n", pos=4, cex=0.8)    #
                                                         #
##########################################################

dev.off()


## Alternative way - 
# Seems easier to follow plus I am not trying to do
# a meta analysis on a model

data <-as.data.frame(cbind(RR, low, high))
raw.overall2 <- as.data.frame(cbind(Study,Year, data))

raw.overall2$logRR <- log(raw.overall2$RR)
raw.overall2$SE <- (log(raw.overall2$high) - log(raw.overall2$low))/(2*1.96)
attach(raw.overall2)
metaplot(logRR, SE, labels=Study, nn=1, logeffect=TRUE, col=meta.colors("black"), xlab="RR of Obesity in high versus low Shannon diversity based on Median", ylab="")
  #Graphical output is not as informative


##Calculate combined P-value
m = 8
T = -2*log(0.608)-2*log(0.214)-2*log(0.236)-2*log(0.439)-2*log(0.575)-2*log(0.376)-2*log(0.022)-2*log(0.64)
T.pval = 1-pchisq(T,m) #P-value = 0.01

##Alternatively can use built in package (I trust this more)
anova.rma(res)
# QM = 5.0973, p-value = 0.0240
```
<br>
This second R chunk shows the code used to generate the RR for obesity and B/F ratio.  

```r
#Load required packages
library(meta)
library(metafor) #This package is used mostly for the more in-depth graph

#Change working directory
setwd("C:/users/marc/Desktop/obesity2")

StudyNo <- c(1, 2, 3, 4, 5, 6, 7, 8)
Study <- c("Baxter", "HMP", "Ross", "Escobar", "Argumugam", "Turnbaugh", "Zupancic", "Wu")
Year <- c("2015", "2011", "2015", "2014", "2010", "2009", "2012", "2012")
Total <- c("n=172", "n=256", "n=63", "n=30", "n=85", "n=146", "n=201", "n=58")
tpos <- c(24, 13, 16, 3, 17, 45, 36, 2) #obese in low BF group
tneg <- c(62, 115, 16, 12, 26, 28, 65, 27) #nonobese in low BF group
cpos <- c(23, 13, 22, 7, 20, 54, 38, 3) #obese in high BF group
cneg <- c(63, 115, 9, 8, 22, 19, 62, 26) #nonobese in high BF group
RR <- c(1.04, 1.00, 0.70, 0.43, 0.83, 0.83, 0.94, 0.67)
low <- c(0.64, 0.48, 0.47, 0.14, 0.51, 0.66, 0.65, 0.12)
high <- c(1.70, 2.07, 1.07, 1.35, 1.35, 1.05, 1.35, 3.70)

numeric.data <-as.data.frame(cbind(tpos, tneg, cpos, cneg))
raw.overall <- as.data.frame(cbind(StudyNo, Study,Year, Total, numeric.data))

dat <- escalc(measure = "RR", ai = tpos, bi = tneg, ci = cpos, di = cneg, data=raw.overall, append=TRUE)
k <- length(dat$StudyNo)
dat.fm <- data.frame(study=factor(rep(1:k, each = 4)))
dat.fm$grp <- factor(rep(c("T", "T", "C", "C"), k), levels = c("T", "C"))
dat.fm$out <- factor(rep(c("+", "-", "+", "-"), k), levels = c("+", "-"))
dat.fm$freq <- with(raw.overall, c(rbind(tpos, tneg, cpos, cneg)))

escalc(out ~ grp | study, weights = freq, data = dat.fm, measure = "RR")


#Used to generate the RE Model estimate based on all the studies
res <- rma(ai=tpos, bi=tneg, ci=cpos, di=cneg, data=raw.overall, measure="RR", slab=paste(Study, Year, Total, sep=", "), method="REML")


#To create a graph visualize the data
forest(res, at=log(c(0.1, 1, 3)), atransf=exp, xlab="RR of Obesity in high versus low B/F ratio based on Median", cex=0.8)
par(font=2) #Switch to bold

# RR Title ################################################
#text(3.2, 3.10, "Relative Risk [95% CI]", pos=2, cex=0.8)#
#large scale (Plot in R studio at full)                 #
#
text(5.5, 9.3, "Relative Risk [95% CI]", pos=2, cex=0.8)  #
#smaller scale (Plot in R studia at normal)             #
###########################################################

# Group Title#############################################
#
text(-6.0, 9.3, "Study, Year, and n", pos=4, cex=0.8)    #
#
##########################################################

dev.off()

##Calculate combined P-value
m = 8
T = -2*log(0.864)-2*log(1.00)-2*log(0.089)-2*log(0.121)-2*log(0.452)-2*log(0.111)-2*log(0.729)-2*log(0.64)
T.pval = 1-pchisq(T,m) #P-value = 0.03155827

##Alternatively can use built in package (I trust this more)
anova.rma(res)
  # QM = 4.8452, p-value = 0.0277
```


#### **Part 3: Pooled Analysis Using ZScore Normalizations**


```r
## Load in libraries to be used as well as directory with the data
library(ggplot2)
library(plyr)
library(reshape2)
library(RColorBrewer)
library(scales)
library(pheatmap)
library(gridExtra)

setwd("C:/Users/marc/Desktop/obesity2/BMI.normalization")

#Load in BMI data
data1 <- read.csv("combined.H.corr.data.csv")
#Load in Turnbaugh data
turn.data <- read.csv("Turnbaugh.normalized.csv")
#Load in Amish data
amish.data <- read.csv("Amish.normalized.csv")

#Amish STD = 0.6010608
amish.ZScore.H <- amish.data$H.corr/0.6010608
hist(amish.ZScore.H)
#Turnbaugh STD = 0.4818751
turn.ZScore.H <- turn.data$H.corr/0.4818751
hist(turn.ZScore.H)
#MetaHit-Den STD = 0.4038558
keepmetahit <- which(data1$dataset == "MetaHit")
metahitdata <- data1[keepmetahit, ]
metahit.ZScore.H <- metahitdata$H.corr/0.4038558
hist(metahit.ZScore.H)
#Korean STD = 0.5400615
keepKorean <- which(data1$dataset == "Korean")
Koreandata <- data1[keepKorean, ]
Korean.ZScore.H <- Koreandata$H.corr/0.5400615
hist(Korean.ZScore.H)
#HMP STD = 0.6774514
keepHMP <- which(data1$dataset == "HMP")
hmpdata <- data1[keepHMP, ]
HMP.ZScore.H <- hmpdata$H.corr/0.6774514
hist(HMP.ZScore.H)
#CCHC STD = 0.5784522
keepCCHC <- which(data1$dataset == "CCHC")
CCHCdata <- data1[keepCCHC, ]
CCHC.ZScore.H <- CCHCdata$H.corr/0.5784522
hist(CCHC.ZScore.H)
#GLNE07 STD = 0.4663871
keepGLNE <- which(data1$dataset == "GLNE")
GLNEdata <- data1[keepGLNE, ]
GLNE.ZScore.H <- GLNEdata$H.corr/0.4663871
hist(GLNE.ZScore.H)
#Escobar STD = 0.5378935
keepescobar <- which(data1$dataset == "Escobar")
escobardata <- data1[keepescobar, ]
Escobar.ZScore.H <- escobardata$H.corr/0.5378935
hist(Escobar.ZScore.H)
#COMBO STD = 0.6870763
keepCOMBO <- which(data1$dataset == "COMBO")
COMBOdata <- data1[keepCOMBO, ]
COMBO.ZScore.H <- COMBOdata$H.corr/0.6870763
hist(COMBO.ZScore.H)

#data1 order Escobar, GLNE, CCHC, HMP, Korean, MetaHit
HZScore <- c(Escobar.ZScore.H, GLNE.ZScore.H, CCHC.ZScore.H, HMP.ZScore.H, Korean.ZScore.H, metahit.ZScore.H, COMBO.ZScore.H)
data1 <- cbind(HZScore, data1)

amish.data <- cbind(amish.ZScore.H, amish.data)
turn.data <- cbind(turn.ZScore.H, turn.data)
rm(Escobar.ZScore.H, GLNE.ZScore.H, CCHC.ZScore.H, HMP.ZScore.H, Korean.ZScore.H, metahit.ZScore.H, amish.ZScore.H, turn.ZScore.H, COMBO.ZScore.H)
rm(keepmetahit, keepCCHC, keepescobar, keepGLNE, keepHMP, keepKorean)

#Create BMI groups
data1$BMI.class[data1$bmi<=24] <- "Lean"
data1$BMI.class[data1$bmi>24 & data1$bmi<30] <- "Overweight"
data1$BMI.class[data1$bmi>=30] <- "Obese"

#Create Obese Yes/No groups
data1$obese[data1$BMI.class=="Lean" | data1$BMI.class=="Overweight"] <- "No"
data1$obese[data1$BMI.class=="Obese"] <- "Yes"

#Create group analysis table to harness full data set
#For ZScore normalization
d1.obese <- data1$obese
t.obese <- as.character(turn.data$obese)
am.obese <- as.character(amish.data$obese)

d1.bmi.class <- data1$BMI.class
t.bmi.class <- as.character(turn.data$BMI.category)

d1.H.Zscore <- data1$HZScore
t.H.Zscore <- turn.data$turn.ZScore.H
am.H.Zscore<- amish.data$amish.ZScore.H

#Combine everything together
comb1.obese <- c(d1.obese, t.obese)
comb2.obese <- c(d1.obese, t.obese, am.obese)
comb1.bmi.class <- c(d1.bmi.class, t.bmi.class)
comb1.H.Zscore <- as.data.frame(c(d1.H.Zscore, t.H.Zscore))
comb2.H.Zscore <- as.data.frame(c(d1.H.Zscore, t.H.Zscore, am.H.Zscore))

group.data.test1 <- cbind(comb1.H.Zscore, comb1.obese, comb1.bmi.class)
group.data.test2 <- cbind(comb2.H.Zscore, comb2.obese)
colnames(group.data.test1) <- c("H.Zscore", "obese", "BMI.class")
colnames(group.data.test2) <- c("H.Zscore", "obese")
#Look at Obese versus non-obese
t.test(H.Zscore ~ obese, data=group.data.test2) #P-value = 0.002792
table(group.data.test2$obese) # No = 693 | Yes = 336
p1 <- ggplot(group.data.test2, aes(obese, H.Zscore)) + geom_boxplot()

#A more in-depth look, breaking down the groups
anova(lm(H.Zscore ~ BMI.class, data=group.data.test1)) #P-value = 0.01783
a2 <- aov(H.Zscore ~ BMI.class, data=group.data.test1)
posthoc <- TukeyHSD(x=a2, 'BMI.class', conf.level=0.95)
  # Obese-Lean (p-value=0.070), Over-Norm (p-value=0.791), Over-Obese (p-value=0.021)

table(group.data.test1$BMI.class) 
  # Lean = 326 | Overweight = 240 | Obese = 262
ddply(group.data.test1, ~BMI.class, summarise, mean=mean(H.Zscore), sd=sd(H.Zscore))
  # Lean = 0.042 +\- 0.940 | Overweight = 0.097 +/- 0.993 | Obese = -0.140 +\- 1.052

#Let's change the order so it makes sense
group.data.test1$BMI.class <- factor(group.data.test1$BMI.class, levels = c("Lean", "Overweight", "Obese"))

p2 <- ggplot(group.data.test1, aes(BMI.class, H.Zscore)) + geom_boxplot()

summary(lm(HZScore ~ bmi, data=data1)) #P-value 0.06558, R2= 0.004977
plot(HZScore ~ bmi, data=data1)
abline(lm(HZScore ~ bmi, data=data1), col="red")

#Make a better graph with ggplot2
p1 <- ggplot(data1, aes(x=bmi, y=HZScore))
p2 <- p1 + geom_point(aes(color=dataset))
p3 <- p2 + scale_colour_brewer(palette="Set1")
p4 <- p3 + geom_smooth(method="lm", se=TRUE)


#Probably want to add the Bacteroidetes / Firmicute ratio
#Large variation in data sets observed so will have to normalize (zscore)
#The data looks funny with either a log numerator with log sd denominator 
#or just sd as the denominator
#Think about using only Zscore but this produces a very large tail

setwd("C:/Users/marc/Desktop/obesity2/BF.normalization")

#Load in data
data2 <- read.csv("combined.BF.corr.csv")

#Create BMI groups
data2$BMI.class[data2$bmi<=24] <- "Lean"
data2$BMI.class[data2$bmi>24 & data2$bmi<30] <- "Overweight"
data2$BMI.class[data2$bmi>=30] <- "Obese"

#Create Obese Yes/No groups
data2$obese[data2$BMI.class=="Lean" | data2$BMI.class=="Overweight"] <- "No"
data2$obese[data2$BMI.class=="Obese"] <- "Yes"

#Amish SD = 0.7990372
am.BF.corr <- as.character(amish.data$BF.ratio.corr)
am.BF.corr <- as.numeric(am.BF.corr) # Value 19 has no value
amish.norm.BF <- am.BF.corr/0.7990372
hist(amish.norm.BF)
#Turnbaugh SD = 0.4578937
turn.norm.BF <- turn.data$BF.corr/0.4578937
hist(turn.norm.BF)
#MetaHit SD = 0.9707577
keepmetahit <- which(data2$dataset == "MetaHit")
metahitdata <- data2[keepmetahit, ]
metahit.norm.BF <- metahitdata$BF.ratio.corr/0.9707577
hist(metahit.norm.BF)
#Korean SD = 0.6048195
keepKorean <- which(data2$dataset == "Korean")
Koreandata <- data2[keepKorean, ]
Korean.norm.BF <- Koreandata$BF.ratio.corr/0.6048195
hist(Korean.norm.BF)
#HMP SD = 8.301864
keepHMP <- which(data2$dataset == "HMP")
hmpdata <- data2[keepHMP, ]
HMP.norm.BF <- hmpdata$BF.ratio.corr/8.301864
hist(HMP.norm.BF)
#CCHC SD = 2.00527
keepCCHC <- which(data2$dataset == "CCHC")
CCHCdata <- data2[keepCCHC, ]
CCHC.norm.BF <- CCHCdata$BF.ratio.corr/2.00527
hist(CCHC.norm.BF)
#GLNE07 SD = 0.4451074
keepGLNE <- which(data2$dataset == "GLNE")
GLNEdata <- data2[keepGLNE, ]
GLNE.norm.BF <- GLNEdata$BF.ratio.corr/0.4451074
hist(GLNE.norm.BF)
#Escobar SD = 0.4232106
keepescobar <- which(data2$dataset == "Escobar")
escobardata <- data2[keepescobar, ]
Escobar.norm.BF <- escobardata$BF.ratio.corr/0.4232106
hist(Escobar.norm.BF)
#COMBO SD = 5.928664
keepCOMBO <- which(data2$dataset == "COMBO")
COMBOdata <- data2[keepCOMBO, ]
COMBO.norm.BF <- COMBOdata$BF.ratio.corr/5.928664
hist(COMBO.norm.BF)

#data1 order Escobar, GLNE, CCHC, HMP, Korean, MetaHit
BFScore <- c(Escobar.norm.BF, GLNE.norm.BF, CCHC.norm.BF, HMP.norm.BF, Korean.norm.BF, metahit.norm.BF, COMBO.norm.BF)
data2 <- cbind(BFScore, data2)

amish.data <- cbind(amish.norm.BF, amish.data)
turn.data <- cbind(turn.norm.BF, turn.data)
rm(Escobar.norm.BF, GLNE.norm.BF, CCHC.norm.BF, HMP.norm.BF, Korean.norm.BF, metahit.norm.BF, amish.norm.BF, turn.norm.BF, COMBO.norm.BF)
rm(keepmetahit, keepCCHC, keepescobar, keepGLNE, keepHMP, keepKorean)

d1.BF.corr <- data2$BFScore
t.BF.corr <- turn.data$turn.norm.BF
am.BF.corr <- amish.data$amish.norm.BF

comb.BF.corr <- as.data.frame(c(d1.BF.corr, t.BF.corr))
group.data.test <- cbind(comb.BF.corr, group.data.test1)
colnames(group.data.test)[1] <- "BF.ratio.corr"

comb2.BF.corr <- as.data.frame(c(d1.BF.corr, t.BF.corr, am.BF.corr))
group.data.test2 <- cbind(comb2.BF.corr, group.data.test2)
colnames(group.data.test2)[1] <- "BF.ratio.corr"

#Look at Obese versus non-obese
t.test(BF.ratio.corr ~ obese, data=group.data.test2) 
  # P-value = 0.05066
table(group.data.test2$obese)
  # No = 693 | Yes = 336
bf1 <- ggplot(group.data.test2, aes(obese, BF.ratio.corr)) + geom_boxplot()

#A more in-depth look, breaking down the groups
anova(lm(BF.ratio.corr ~ BMI.class, data=group.data.test)) 
  # P-value = 0.0227
a2 <- aov(BF.ratio.corr ~ BMI.class, data=group.data.test)
posthoc <- TukeyHSD(x=a2, 'BMI.class', conf.level=0.95)
  # Obese-Norm (p-value=0.026) | Over-Norm (p-value=0.117), Over-Obese (p-value=0.863)
table(group.data.test$BMI.class)
  # Lean = 326 | Overweight = 240 | Obese = 262
ddply(group.data.test, ~BMI.class, summarise, mean=mean(BF.ratio.corr), sd=sd(BF.ratio.corr))
  # Lean = -0.361 +\- 1.431 | Overweight = -0.695 +/- 2.032 | Obese = -0.787 +\- 2.461
bf2 <- ggplot(group.data.test, aes(BMI.class, BF.ratio.corr)) + geom_boxplot()


summary(lm(BFScore ~ bmi, data=data2)) #P-value 0.07859, R2= 0.004543
plot(BFScore ~ bmi, data=data2)
abline(lm(BFScore ~ bmi, data=data2), col="red")

#Make a better graph with ggplot2
b1 <- ggplot(data2, aes(x=bmi, y=BFScore))
b2 <- b1 + geom_point(aes(color=dataset))
b3 <- b2 + scale_colour_brewer(palette="Set1")
b4 <- b3 + geom_smooth(method="lm", se=TRUE)


##Run Zscore normalized Bacter and Firm overall analysis
#data1 order Escobar, GLNE, CCHC, HMP, Korean, MetaHit
d1.bacter.corr <- data2$Zbacter
d1.firm.corr <- data2$Zfirm
d1.zbf.corr <- data2$ZBF
t.bacter.corr <- turn.data$Zbacter
t.firm.corr <- turn.data$Zfirm
t.zbf.corr <- turn.data$ZBF
am.bacter.corr <- amish.data$Zbacter
am.firm.corr <- amish.data$Zfirm
am.zbf.corr <- amish.data$ZBF

#Bacteroidetes Datasets
comb.bacter.corr <- as.data.frame(c(d1.bacter.corr, t.bacter.corr))
group.data.test <- cbind(comb.bacter.corr, group.data.test)
colnames(group.data.test)[1] <- "Zbacter"

comb2.bacter.corr <- as.data.frame(c(d1.bacter.corr, t.bacter.corr, am.bacter.corr))
group.data.test2 <- cbind(comb2.bacter.corr, group.data.test2)
colnames(group.data.test2)[1] <- "Zbacter"

#Firmicutes Datasets
comb.firm.corr <- as.data.frame(c(d1.firm.corr, t.firm.corr))
group.data.test <- cbind(comb.firm.corr, group.data.test)
colnames(group.data.test)[1] <- "Zfirm"

comb2.firm.corr <- as.data.frame(c(d1.firm.corr, t.firm.corr, am.firm.corr))
group.data.test2 <- cbind(comb2.firm.corr, group.data.test2)
colnames(group.data.test2)[1] <- "Zfirm"

#Bacteroidetes/Firmicutes Datasets
comb.zbf.corr <- as.data.frame(c(d1.zbf.corr, t.zbf.corr))
group.data.test <- cbind(comb.zbf.corr, group.data.test)
colnames(group.data.test)[1] <- "zbf"

comb2.zbf.corr <- as.data.frame(c(d1.zbf.corr, t.zbf.corr, am.zbf.corr))
group.data.test2 <- cbind(comb2.zbf.corr, group.data.test2)
colnames(group.data.test2)[1] <- "zbf"

###################################Bacteroidetes#######################################
#Look at Obese versus non-obese
t.test(Zbacter ~ obese, data=group.data.test2) 
  # P-value = 0.3161
table(group.data.test2$obese)
  # No = 693 | Yes = 336
bacter1 <- ggplot(group.data.test2, aes(obese, Zbacter)) + geom_boxplot()

#A more in-depth look, breaking down the groups
anova(lm(Zbacter ~ BMI.class, data=group.data.test)) #P-value = 0.0957
a2 <- aov(Zbacter ~ BMI.class, data=group.data.test)
posthoc <- TukeyHSD(x=a2, 'BMI.class', conf.level=0.95)
  # Obese-Norm (p-value=0.979) | Over-Norm (p-value=0.151) | Over-Obese (p-value=0.125)
table(group.data.test$BMI.class)
  # Lean = 326 | Overweight = 240 | Obese = 262
ddply(group.data.test, ~BMI.class, summarise, mean=mean(Zbacter), sd=sd(Zbacter))
  # Lean = 0.041 +/- 0.966 | Overweight = -0.117 +/- 1.000 | Obese = 0.057 +/- 1.02

bacter2 <- ggplot(group.data.test, aes(BMI.class, Zbacter)) + geom_boxplot()


summary(lm(Zbacter ~ bmi, data=data2)) #P-value 0.7168, R2= 0.0001936
plot(Zbacter ~ bmi, data=data2)
abline(lm(Zbacter ~ bmi, data=data2), col="red")

#Make a better graph with ggplot2
b1 <- ggplot(data2, aes(x=bmi, y=Zbacter))
b2 <- b1 + geom_point(aes(color=dataset))
b3 <- b2 + scale_colour_brewer(palette="Set1")
b4 <- b3 + geom_smooth(method="lm", se=TRUE)

###################################Firmicutes#######################################
#Look at Obese versus non-obese
t.test(Zfirm ~ obese, data=group.data.test2) #P-value = 0.8298
table(group.data.test2$obese)
  # No = 693 | Yes = 336
firm1 <- ggplot(group.data.test2, aes(obese, Zfirm)) + geom_boxplot()

#A more in-depth look, breaking down the groups
anova(lm(Zfirm ~ BMI.class, data=group.data.test)) 
  #P-value = 0.05521
a2 <- aov(Zfirm ~ BMI.class, data=group.data.test)
posthoc <- TukeyHSD(x=a2, 'BMI.class', conf.level=0.95)
  # Obese-Norm (p-value=0.976), Over-Norm (p-value=0.063), Over-Obese (p-value=0.1252)
table(group.data.test$BMI.class)
  # Lean = 326 | Overweight = 240 | Obese = 262
ddply(group.data.test, ~BMI.class, summarise, mean=mean(Zfirm), sd=sd(Zfirm))
  # Lean = -0.061 +/- 0.959 | Overweight = 0.130 +/- 1.018 | Obese = -0.043 +/- 1.013

firm2 <- ggplot(group.data.test, aes(BMI.class, Zfirm)) + geom_boxplot()


summary(lm(Zfirm ~ bmi, data=data2)) #P-value 0.8682, R2= 4.06x10-5
plot(Zfirm ~ bmi, data=data2)
abline(lm(Zfirm ~ bmi, data=data2), col="red")

#Make a better graph with ggplot2
b1 <- ggplot(data2, aes(x=bmi, y=Zfirm))
b2 <- b1 + geom_point(aes(color=dataset))
b3 <- b2 + scale_colour_brewer(palette="Set1")
b4 <- b3 + geom_smooth(method="lm", se=TRUE)

###################################Bacteroidetes/Firmicutes#######################################
#Look at Obese versus non-obese
t.test(zbf ~ obese, data=group.data.test2) #P-value = 0.4167
table(group.data.test2$obese)
  # No = 693 | Yes = 336
zbf1 <- ggplot(group.data.test2, aes(obese, zbf)) + geom_boxplot()

#A more in-depth look, breaking down the groups
anova(lm(zbf ~ BMI.class, data=group.data.test)) 
  # P-value = 0.4769
a2 <- aov(zbf ~ BMI.class, data=group.data.test)
posthoc <- TukeyHSD(x=a2, 'BMI.class', conf.level=0.95)
  # Obese-Norm (p-value=0.892) | Over-Norm (p-value=0.692) | Over-Obese (p-value=0.452)
table(group.data.test$BMI.class)
  # Lean = 326 | Overweight = 240 | Obese = 262
ddply(group.data.test, ~BMI.class, summarise, mean=mean(zbf), sd=sd(zbf))
  # Lean = 0.0081 +/- 0.954 | Overweight = -0.064 +/- 0.951 | Obese = 0.046 +/- 1.084
zbf2 <- ggplot(group.data.test, aes(BMI.class, zbf)) + geom_boxplot()


summary(lm(ZBF ~ bmi, data=data2)) #P-value 0.7118, R2= 0.0002008
plot(ZBF ~ bmi, data=data2)
abline(lm(ZBF ~ bmi, data=data2), col="red")

#Make a better graph with ggplot2
b1 <- ggplot(data2, aes(x=bmi, y=ZBF))
b2 <- b1 + geom_point(aes(color=dataset))
b3 <- b2 + scale_colour_brewer(palette="Set1")
b4 <- b3 + geom_smooth(method="lm", se=TRUE)
```





##### **References:**  





