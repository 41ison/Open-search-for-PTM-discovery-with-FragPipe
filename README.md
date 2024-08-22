# Open search for PTM discovery with FragPipe

This is a quick guide to analyze the results of an open search for PTM discovery with FragPipe v22. The analysis is based on the output file generated by PTM-Shepherd.

Please, checke the [FragPipe](https://fragpipe.nesvilab.org/docs/tutorial_open.html/) documentation for more information about the open search analysis.
Below you find some descriptions of what we are doing in this script with the information provided by the FragPipe documentation.

`global.profile.tsv` reports the most prominent features from PTM-Shepherd analysis of mass shifts observed from FDR-filtered open search results. Each row corresponds to a different detected mass shift, thus not all PSMs will be represented in this table. Please note that mass shifts are annotated based on UniMod mapping, thus they are not definitive chemical identities and should be used as a starting point along with localization and amino acid enrichment information. Unless otherwise indicated, values are summed from all datasets in the analysis.

## The following plot is the idea of what you will extract from the PTM-Shepherd output:
![alt text](https://github.com/41ison/Open-search-for-PTM-discovery-with-FragPipe/edit/main/OpenSearch_modification.png "Mapped mass-shift")

## The following R packages are required to run this script:

```{r}
library(tidyverse)
library(janitor)
library(here)
library(pals)

theme_set(theme_minimal())
```

## Load the data from PTM Sheperd output
Once your PTM-Shepherd analysis is done, you can put this script in the same foledr as the file called `global.profile.tsv`, then you can run the following code to load the data.

```{r}
global_profile <- read_tsv("global.profile.tsv") %>%
  clean_names()

View(global_profile)
```

# Extracting the information from mapped masses
1. `AA1_psm_count` weighted number of PSMs where the mass shift localized to AA1. Shifts localizing to multiple residues are divided by the number of localized residues in the spectra, so this is an estimated number of PSMs localized to a particular residue.

2. `AA1` amino acid/residue most enriched (most likely to harbor the mass shift) compared to other residues.

3. `mapped_mass_1` primary modification annotation derived from Unimod, all isobaric modifications listed and separated by “/”.

```{r}
OpenSearch_modification <- global_profile %>%
    ggplot(aes(x = aa1_psm_count, y = mapped_mass_1, 
                size = aa1_psm_count, color = aa1)) +
    geom_segment(aes(xend = 0, yend = mapped_mass_1),
                size = 0.5, show.legend = FALSE) +
    geom_point() +
    guides(color = guide_legend(override.aes = list(size = 8)),
                size = "none") +
    geom_text(aes(label = aa1_psm_count), 
                hjust = -0.5, vjust = 0.5, size = 5,
                show.legend = FALSE) +
    geom_vline(xintercept = 0, linetype = "dashed", color = "black") +
    labs(title = "Open search modification",
                y = "Mapped mass",
                x = "Weighted number of PSM",
                color = "Amino Acid") +
    theme(legend.position = "bottom",
                legend.title.position = "top",
                legend.title = element_text(hjust = 0.5,
                                face = "bold", size = 15),
                plot.title = element_text(hjust = 0.5,
                                face = "bold", size = 20),
                text = element_text(size = 15)) +
    scale_color_manual(values = pals::cols25(22))

# See your plot
print(OpenSearch_modification)

# Save your plot
ggsave("OpenSearch_modification.png", 
    plot = OpenSearch_modification,
    width = 20, height = 10, bg = "white", 
    units = "in", dpi = 300)
```
