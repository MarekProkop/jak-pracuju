---
title: "Jak se zbavit azbuky"
description: |
  Do vyhledávacích dotazů nebo jiných dat se občas dostane ruština či jiný jazyk psaný azbukou a dělá tam nepořádek. Tohle je rýchlý návod, jak se textů v azbuce zbavit, nebo je naopak najít.
author:
  - name: Marek Prokop
    affiliation: PROKOP software s.r.o.
    affiliation_url: https://www.prokopsw.cz/cs
date: 2022-04-23
output:
  distill::distill_article:
    self_contained: false
    toc: true
draft: true
---

MacOS je v tomhle snad lepší, ale na Windows se v R resp. RStudiu blbě zobrazuje azbuka (cyrilice) a navíc v typických situacích, třeba ve vyhledávacích dotazech, ani texty v azbuce nechci.





## Rychlý příklad

Mám tahle data:


```r
strings <- c(
  "prague weather",
  "weather prague",
  "погода прага",
  "prague airport",
  "прага киев самолет",
  "vietnam restaurant prague"
)

df <- tibble(text = strings, number = sample(1000:5000, 6))
```

Vektor se v RStudiu zobrazuje správně v R Markdownu i na konsoli, ale v HTML, které vygeneruje [knitr](https://yihui.org/knitr/), je azbuka zmršená.


```r
print(strings)
```

```
## [1] "prague weather"                                                                                                                    
## [2] "weather prague"                                                                                                                    
## [3] "<U+043F><U+043E><U+0433><U+043E><U+0434><U+0430> <U+043F><U+0440><U+0430><U+0433><U+0430>"                                         
## [4] "prague airport"                                                                                                                    
## [5] "<U+043F><U+0440><U+0430><U+0433><U+0430> <U+043A><U+0438><U+0435><U+0432> <U+0441><U+0430><U+043C><U+043E><U+043B><U+0435><U+0442>"
## [6] "vietnam restaurant prague"
```

Dataframe je zmršený i v RStudiu.


```r
df |> 
  mutate(text = iconv(text, to = "utf-8"))
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["text"],"name":[1],"type":["chr"],"align":["left"]},{"label":["number"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"prague weather","2":"4100"},{"1":"weather prague","2":"3617"},{"1":"<U+043F><U+043E><U+0433><U+043E><U+0434><U+0430> <U+043F><U+0440><U+0430><U+0433><U+0430>","2":"2651"},{"1":"prague airport","2":"4970"},{"1":"<U+043F><U+0440><U+0430><U+0433><U+0430> <U+043A><U+0438><U+0435><U+0432> <U+0441><U+0430><U+043C><U+043E><U+043B><U+0435><U+0442>","2":"2025"},{"1":"vietnam restaurant prague","2":"2596"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Jak texty v azbuce odstranit

Stačí třída v regulárních výrazech. Konkrétně v případě vektoru:


```r
strings[!stringi::stri_detect_charclass(strings, "[\\p{Cyrillic}]")]
```

```
## [1] "prague weather"                                                                                                                    
## [2] "weather prague"                                                                                                                    
## [3] "<U+043F><U+043E><U+0433><U+043E><U+0434><U+0430> <U+043F><U+0440><U+0430><U+0433><U+0430>"                                         
## [4] "prague airport"                                                                                                                    
## [5] "<U+043F><U+0440><U+0430><U+0433><U+0430> <U+043A><U+0438><U+0435><U+0432> <U+0441><U+0430><U+043C><U+043E><U+043B><U+0435><U+0442>"
## [6] "vietnam restaurant prague"
```

A v případě dataframu:


```r
df |> 
  filter(grepl("[\\p{IsCyrillic}]", strings))
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["text"],"name":[1],"type":["chr"],"align":["left"]},{"label":["number"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"prague weather","2":"4100"},{"1":"weather prague","2":"3617"},{"1":"prague airport","2":"4970"},{"1":"<U+043F><U+0440><U+0430><U+0433><U+0430> <U+043A><U+0438><U+0435><U+0432> <U+0441><U+0430><U+043C><U+043E><U+043B><U+0435><U+0442>","2":"2025"},{"1":"vietnam restaurant prague","2":"2596"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Ještě ke zobrazování

Jen pro zajímavost a kontext, `knitr::kable()` zobrazuje azbuku blbě v R Markdownu i na konzoli.


```r
df |> knitr::kable()
```



|text                                                                                                                               | number|
|:----------------------------------------------------------------------------------------------------------------------------------|------:|
|prague weather                                                                                                                     |   4100|
|weather prague                                                                                                                     |   3617|
|<U+043F><U+043E><U+0433><U+043E><U+0434><U+0430> <U+043F><U+0440><U+0430><U+0433><U+0430>                                          |   2651|
|prague airport                                                                                                                     |   4970|
|<U+043F><U+0440><U+0430><U+0433><U+0430> <U+043A><U+0438><U+0435><U+0432> <U+0441><U+0430><U+043C><U+043E><U+043B><U+0435><U+0442> |   2025|
|vietnam restaurant prague                                                                                                          |   2596|

Reactable zobrazuje azbuku v R Markdownu dobře, ale blbě se zknituje.


```r
df |> reactable()
```

```
## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.
```

```
## Error in path.expand(path): invalid 'path' argument
```

Totéž xtable -- dobře v R Mardownu, knitr špatně.


```r
df |> qflextable()
```

```{=html}
<div class="tabwid"><style>.cl-f84b998a{}.cl-f841edcc{font-family:'Arial';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-f84214c8{margin:0;text-align:left;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-f84214c9{margin:0;text-align:right;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-f84262c0{width:57.8pt;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c1{width:859.7pt;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c2{width:859.7pt;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c3{width:57.8pt;background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c4{width:57.8pt;background-color:transparent;vertical-align: middle;border-bottom: 2pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c5{width:859.7pt;background-color:transparent;vertical-align: middle;border-bottom: 2pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c6{width:57.8pt;background-color:transparent;vertical-align: middle;border-bottom: 2pt solid rgba(102, 102, 102, 1.00);border-top: 2pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-f84262c7{width:859.7pt;background-color:transparent;vertical-align: middle;border-bottom: 2pt solid rgba(102, 102, 102, 1.00);border-top: 2pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table class='cl-f84b998a'>
```

```{=html}
<thead><tr style="overflow-wrap:break-word;"><td class="cl-f84262c7"><p class="cl-f84214c8"><span class="cl-f841edcc">text</span></p></td><td class="cl-f84262c6"><p class="cl-f84214c9"><span class="cl-f841edcc">number</span></p></td></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-f84262c1"><p class="cl-f84214c8"><span class="cl-f841edcc">prague weather</span></p></td><td class="cl-f84262c0"><p class="cl-f84214c9"><span class="cl-f841edcc">4,100</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-f84262c1"><p class="cl-f84214c8"><span class="cl-f841edcc">weather prague</span></p></td><td class="cl-f84262c0"><p class="cl-f84214c9"><span class="cl-f841edcc">3,617</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-f84262c2"><p class="cl-f84214c8"><span class="cl-f841edcc">&lt;U+043F&gt;&lt;U+043E&gt;&lt;U+0433&gt;&lt;U+043E&gt;&lt;U+0434&gt;&lt;U+0430&gt; &lt;U+043F&gt;&lt;U+0440&gt;&lt;U+0430&gt;&lt;U+0433&gt;&lt;U+0430&gt;</span></p></td><td class="cl-f84262c3"><p class="cl-f84214c9"><span class="cl-f841edcc">2,651</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-f84262c1"><p class="cl-f84214c8"><span class="cl-f841edcc">prague airport</span></p></td><td class="cl-f84262c0"><p class="cl-f84214c9"><span class="cl-f841edcc">4,970</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-f84262c2"><p class="cl-f84214c8"><span class="cl-f841edcc">&lt;U+043F&gt;&lt;U+0440&gt;&lt;U+0430&gt;&lt;U+0433&gt;&lt;U+0430&gt; &lt;U+043A&gt;&lt;U+0438&gt;&lt;U+0435&gt;&lt;U+0432&gt; &lt;U+0441&gt;&lt;U+0430&gt;&lt;U+043C&gt;&lt;U+043E&gt;&lt;U+043B&gt;&lt;U+0435&gt;&lt;U+0442&gt;</span></p></td><td class="cl-f84262c3"><p class="cl-f84214c9"><span class="cl-f841edcc">2,025</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-f84262c5"><p class="cl-f84214c8"><span class="cl-f841edcc">vietnam restaurant prague</span></p></td><td class="cl-f84262c4"><p class="cl-f84214c9"><span class="cl-f841edcc">2,596</span></p></td></tr></tbody></table></div>
```

## Session info


```r
print(sessionInfo())
```

```
## R version 4.1.3 (2022-03-10)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 19043)
## 
## Matrix products: default
## 
## locale:
## [1] LC_COLLATE=Czech_Czechia.1250  LC_CTYPE=Czech_Czechia.1250   
## [3] LC_MONETARY=Czech_Czechia.1250 LC_NUMERIC=C                  
## [5] LC_TIME=Czech_Czechia.1250    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
##  [1] flextable_0.7.0 reactable_0.2.3 stringi_1.7.6   forcats_0.5.1   stringr_1.4.0  
##  [6] dplyr_1.0.8     purrr_0.3.4     readr_2.1.2     tidyr_1.2.0     tibble_3.1.6   
## [11] ggplot2_3.3.5   tidyverse_1.3.1 usethis_2.1.5  
## 
## loaded via a namespace (and not attached):
##  [1] httr_1.4.2           jsonlite_1.8.0       warp_0.2.0           modelr_0.1.8        
##  [5] assertthat_0.2.1     distributional_0.3.0 highr_0.9            fabletools_0.3.2    
##  [9] cellranger_1.1.0     tsibble_1.1.1        yaml_2.3.5           gdtools_0.2.4       
## [13] pillar_1.7.0         backports_1.4.1      glue_1.6.2           uuid_1.0-4          
## [17] digest_0.6.29        rvest_1.0.2          colorspace_2.0-3     htmltools_0.5.2     
## [21] reactR_0.4.4         pkgconfig_2.0.3      broom_0.7.12         haven_2.4.3         
## [25] webshot_0.5.2        scales_1.1.1         slider_0.2.2         officer_0.4.2       
## [29] distill_1.3          tzdb_0.3.0           downlit_0.4.0        generics_0.1.2      
## [33] farver_2.1.0         ellipsis_0.3.2       cachem_1.0.6         withr_2.5.0         
## [37] cli_3.2.0            magrittr_2.0.3       crayon_1.5.1         readxl_1.4.0        
## [41] memoise_2.0.1        evaluate_0.15        fs_1.5.2             fansi_1.0.3         
## [45] anytime_0.3.9        xml2_1.3.3           tools_4.1.3          data.table_1.14.2   
## [49] feasts_0.2.2         hms_1.1.1            lifecycle_1.0.1      munsell_0.5.0       
## [53] reprex_2.0.1         zip_2.2.0            compiler_4.1.3       systemfonts_1.0.4   
## [57] rlang_1.0.2          grid_4.1.3           rstudioapi_0.13      htmlwidgets_1.5.4   
## [61] crosstalk_1.2.0      base64enc_0.1-3      rmarkdown_2.13       gtable_0.3.0        
## [65] DBI_1.1.2            R6_2.5.1             lubridate_1.8.0      knitr_1.38          
## [69] fastmap_1.1.0        utf8_1.2.2           Rcpp_1.0.8.3         vctrs_0.4.0         
## [73] dbplyr_2.1.1         tidyselect_1.1.2     xfun_0.30
```

