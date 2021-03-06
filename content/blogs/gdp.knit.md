---
categories:
- ""
- ""
date: "2017-10-31T21:28:43-05:00"
description: ""
draft: false
keywords: ""
slug: gdp
title: GDP
---





At the risk of oversimplifying things, the main components of gross domestic product, GDP are personal consumption (C), business investment (I), government spending (G) and net exports (exports - imports). Read more about GDP and the different approaches in calculating at the [Wikipedia GDP page](https://en.wikipedia.org/wiki/Gross_domestic_product).

The GDP data we will look at is from the [United Nations' National Accounts Main Aggregates Database](https://unstats.un.org/unsd/snaama/Downloads), which contains estimates of total GDP and its components for all countries from 1970 to today. We will look at how GDP and its components have changed over time, and compare different countries and how much each component contributes to that country's GDP. 



```r
UN_GDP_data  <-  read_excel(here::here("data", "Download-GDPconstant-USD-countries.xls"), # Excel filename
                sheet="Download-GDPconstant-USD-countr", # Sheet name
                skip=2) # Number of rows to skip
```

 The first thing we do is to tidy the data, as it is in wide format and we must make it into long, tidy format. All figures are expressed in billions (divide values by `1e9`, or $10^9$), and we rename the indicators into something shorter.



```r
tidy_GDP_data<-UN_GDP_data %>% 
  pivot_longer(cols=c(4:51),names_to="year",values_to="figure") %>% 
  mutate(figure=figure/1e9) 
tidy_GDP_data<-tidy_GDP_data %>% 
  clean_names() %>% 
  mutate(Indicator_Name=case_when(
    tidy_GDP_data$IndicatorName=="Exports of goods and services"~"Exports",
    tidy_GDP_data$IndicatorName=="Household consumption expenditure (including Non-profit institutions serving households)"~"Household expeniture",
    tidy_GDP_data$IndicatorName=="Imports of goods and services"~"Imports",
    tidy_GDP_data$IndicatorName=="General government final consumption expenditure"~"Government expenditure",
    tidy_GDP_data$IndicatorName=="Gross capital formation"~"Gross capital formation",
    TRUE~ "Other Data"))

 # Let us compare GDP components for these 3 countries
country_list <- c("United States","India", "Germany")
```


```r
three_country<-tidy_GDP_data %>% filter(country %in% country_list)%>% 
  filter(Indicator_Name %in% c("Gross capital formation", "Exports","Household expeniture","Imports","Government expenditure")) 
  
  
ggplot(three_country,aes(x=year,y=figure,color=Indicator_Name))+
  geom_point(size=0.5)+
  geom_line(aes(group=Indicator_Name))+
  facet_wrap(~country)+
  theme(legend.position = "right")+
  guides(colour = guide_legend(nrow = 5))+
  theme(axis.text.x = element_text(angle = 90))+
  scale_x_discrete(breaks=c(1970,1980,1990,2000,2010))+ 
  labs(title="GDP components over time", subtitle="In constant 2010 USD", x="",
       y="Billion US$")
```

<img src="gdp_files/figure-html/unnamed-chunk-1-1.png" width="672" />


We recall that GDP is the sum of Household Expenditure (Consumption *C*), Gross Capital Formation (business investment *I*), Government Expenditure (G) and Net Exports (exports - imports). Even though there is an indicator `Gross Domestic Product (GDP)` in your dataframe, we calculate it given its components discussed above.

> What is the % difference between what we calculated as GDP and the GDP figure included in the dataframe?


```r
export<-three_country %>% 
  filter(Indicator_Name=="Exports")

Import<-three_country %>% 
  filter(Indicator_Name=="Imports")

net_export_join<-left_join(export,Import, by=c("country","year"))

net_export<-net_export_join%>% 
  mutate(net_export=net_export_join$figure.x-net_export_join$figure.y)%>% 
  select(year,country,net_export)

G<-three_country %>% 
  filter(Indicator_Name=="Government expenditure") %>%
  select(year,country,figure) %>% 
  rename(government=figure)

C<-three_country %>% 
  filter(Indicator_Name=="Household expeniture") %>%
  select(year,country,figure) %>% 
  rename(Consumption=figure)

I<-three_country %>% 
  filter(Indicator_Name=="Gross capital formation") %>%
  select(year,country,figure) %>% 
  rename(Investment=figure)

GDP<-left_join(G,net_export,by=c("year","country")) %>% 
  left_join(C,by=c("year","country")) %>%
  left_join(I,net_export,by=c("year","country")) %>% 
  mutate(gdp=government+Investment+Consumption+net_export)

GDP_given<- tidy_GDP_data %>% 
  filter(country %in% country_list) %>% 
  filter(indicator_name=="Gross Domestic Product (GDP)") %>% 
  select(year,country,figure) %>%
  rename(GDP_given=figure)

gdp_compared<-left_join(GDP,GDP_given,by=c("country","year")) %>% 
  mutate(difference=(gdp-GDP_given)/GDP_given*100)
```




```r
gdp_proportion<-gdp_compared %>% 
  mutate(goverment_prop=government/gdp*100,
         net_export_prop=net_export/gdp*100,
         Consumption_prop=Consumption/gdp*100,
         Investment_prop=Investment/gdp*100) %>% 
  select(year,country,goverment_prop,net_export_prop,Investment_prop,Consumption_prop) %>% 
  pivot_longer(cols=c(3:6),names_to = "GDP_component",values_to = "proportion")


ggplot(gdp_proportion,aes(x=year,y=proportion,color=GDP_component))+
  geom_point(size=0.5)+
  geom_line(aes(group=GDP_component))+
  facet_wrap(~country)+
  theme(legend.position = "right")+
  guides(colour = guide_legend(nrow = 5))+
  theme(axis.text.x = element_text(angle = 90))+
  scale_x_discrete(breaks=c(1970,1980,1990,2000,2010))+ 
  labs(title="GDP and its breakdown at constant 2010 prices in US Dollars", x="",y="proportion")
```

<img src="gdp_files/figure-html/unnamed-chunk-3-1.png" width="672" />



> The last chart tells us the changes of 4 GDP components from 1970 to 2010 of Germany,India and the US. We could also tell the proportion of GDP components. For all three countries, the consumption ocupies the most proportion of the total GDP. Germany's consumption proportion remained the same throughout 40 years, while the consumption proportion of the India decreased significantly, and the reason of that was that the investment proportion of the GDP went up during that period. The consumption proportion of the US went up smoothly.


> As for the net export, for all three countries, the net export all ocupied the least proportion of the total GDP. Specifically, the net export of Germany grew up since 2000, which meant that the exports of Germany began to be greater than the imports. On the contrary, the net export of the US gradually went down, meaning that its imports was still greater than the exports.

> For the government spending proportion, both India and Germany's proportion remained the same during the 40 years, while the US decreased slightly, and it was just in the decreasing period of the government spending, the investment proportion of the US began to increase.
