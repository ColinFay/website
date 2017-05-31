---

title: '1 dataset, 10 visualisations — retour de La Fabrique de l’info'
author: colin_fay
post_date: 2017-03-12 18:00:45
categories:r-blog-fr
layout: single
permalink: /1-dataset-10-visualisations/
published: true
---
## Retour sur l’atelier dataviz de la Fabrique de l’info !<!--more-->

## La fabrique de l’info
<a href="https://colinfay.github.io/wp-content/uploads/2017/03/IMG_2771.jpg"><img class="size-medium wp-image-1412 alignleft" src="https://colinfay.github.io/wp-content/uploads/2017/03/IMG_2771-300x225.jpg" alt="" width="300" height="225" /></a>

Lors de cette journée d’ateliers et de conférences, j’ai animé un atelier sur la thématique de la dataviz. Le message ? Une visualisation de données, autant qu'un texte, relève d’un choix éditorial — et un dataset peut dire énormément de choses différentes en fonction du message que l'on souhaite passer.
<p style="text-align: center;"><a href="https://github.com/ColinFay/conf/blob/master/fabrique-info/La%20fabrique%20de%20l'info.pdf">Retrouvez les slides de la présentation ici !</a></p>
Pour cela, j’ai notamment pris l’exemple d’un <a href="https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/">jeu de données</a>, que j'ai décliné en 10 visualisations. Voici comment les réaliser avec R.

## 1 jeu de données, 10 visualisations
_Notes_ Les dataviz ont été crées avec le support des `databzhtools` que vous pouvez télécharger ici : <a class="uri" href="https://github.com/DataBzh/data-bzh-tools">https://github.com/DataBzh/Data-bzh-tools</a>
Pour reproduire exactement ces visualisations, téléchargez ces outils, et charger les via : `source("data-bzh-tools-master/main.R")`

```{r} 
#Charger les données
library(tidyverse)
source(file = "data-bzh-tools-master/main.R")
prim <- read_csv2("https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/download/?format=csv&amp;timezone=Europe/Berlin&amp;use_labels_for_header=true")
prim$Année <- paste0("01-01-",prim$Année) %>%
  lubridate::dmy()
```
### Visualisation d’effectifs
```{r} 
prim %>%
  group_by(Année) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(Année, somme)) + 
  geom_bar(stat = "identity", fill = databzh$colour1) + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```
 <a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot06.jpeg"><img class="aligncenter size-large wp-image-1393" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot06-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Proportions par secteur
```{r} 
prim %>%
  group_by(Année, `Secteur disciplinaire`) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(Année, somme, fill = `Secteur disciplinaire`)) + 
  scale_fill_manual(values = databzh$colours) +
  geom_bar(stat = "identity", position = "fill") + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot07.jpeg"><img class="aligncenter size-large wp-image-1394" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot07-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Par secteur, en valeur absolue
```{r} 
prim %>%
  group_by(Année, Bénéficiaires, `Secteur disciplinaire`) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(Année,somme, group = `Secteur disciplinaire`, col = `Secteur disciplinaire`)) + 
  geom_point() +
  scale_color_manual(values = databzh$colours) +
  facet_grid(~`Secteur disciplinaire`) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot12.jpeg"><img class="aligncenter size-large wp-image-1395" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot12-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Carte de France
```{r} 
#states <- map_data("france")
prim2 <- prim %>%
  separate(`Géo-localisation`, into = c("lon","lat"), sep =',') %>%
  group_by(lon, lat) %>%
  summarise(somme = sum(Bénéficiaires))
ggplot(states, aes(long,lat, group=group)) + 
  geom_polygon(fill = "#e4e4e4") + 
  coord_map() +
  geom_path(data=states, aes(long, lat, group=group, fill=NULL), color="grey50") +
  geom_point(data = prim2, aes(x = as.numeric(lat), y = as.numeric(lon), group = NULL,size = somme, col = databzh$colour3)) + 
  scale_color_manual(values = palette) + 
  scale_size(range = c(1,10)) +
  xlim(range(range(states$long))) +
  ylim(range(range(states$lat))) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot20.jpeg"><img class="aligncenter size-large wp-image-1396" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot20-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Boxplot
```{r} 
ggplot(prim, aes(Région, Bénéficiaires)) + 
  geom_boxplot(col = databzh$colour4) +
  coord_flip() +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot18.jpeg"><img class="aligncenter size-large wp-image-1397" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot18-1024x512.jpeg" alt="" width="840" height="420" /></a>
## Dotplot
```{r} 
prim %>%
  group_by(Région) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(reorder(Région, somme), somme)) +
  geom_point(col = databzh$colour1, size=4) + 
  coord_flip() + 
  xlab("")+
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot15.jpeg"><img class="aligncenter size-large wp-image-1399" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot15-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Dotplot bis
```{r} 
prim %>%
  group_by(Établissement) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  na.omit() %>%
  ggplot(aes(reorder(Établissement, somme), somme)) +
  geom_point(col = databzh$colour3, size=4) + 
  coord_flip() + 
  xlab("")+
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot17.jpeg"><img class="aligncenter size-large wp-image-1400" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot17-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Histogramme
```{r} 
prim %>%
  group_by(Établissement) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  na.omit() %>%
  ggplot(aes(somme)) +
  geom_histogram(fill = databzh$colour3) + 
  xlab("Bénéficiaire")+
  ylab("Nombre d'établissements dans la tranche") +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot19.jpeg"><img class="aligncenter size-large wp-image-1401" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot19-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Ligne
```{r} 
prim %>%
  group_by(Année, `Groupe de corps`) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(Année, somme, group =`Groupe de corps`, col =`Groupe de corps`)) + 
  geom_line(stat = "identity", size = 3) +
  scale_color_manual(values = databzh$colours) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```

<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot14.jpeg"><img class="aligncenter size-large wp-image-1402" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot14-1024x512.jpeg" alt="" width="840" height="420" /></a>

### Barplot
```{r} 
prim %>%
  group_by(Année, Bénéficiaires, Sexe) %>%
  summarise(somme = sum(Bénéficiaires))%>%
  ggplot(aes(Année, somme)) + 
  geom_bar(stat = "identity", fill = databzh$colour2) +
  scale_fill_manual(values = databzh$colours) +
  facet_grid(Sexe~.) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot09.jpeg"><img class="aligncenter size-large wp-image-1403" src="https://colinfay.github.io/wp-content/uploads/2017/03/Rplot09-1024x512.jpeg" alt="" width="840" height="420" /></a>

