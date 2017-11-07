# Noncompartmental Analysis by R

진행 중인 작업.

## NonCompart 0.3.3 - 2017-08-14 (local)

- NonCompart::tblNCA()

``` r
make gitbook
```



# 서론 {#intro}

약동학 분야에서 가장 간단하고도 객관적이며 널리 쓰이는 방법은 비구획분석 (Non-compartmental analysis, NCA)입니다. 
*미국의 FDA (Food and Drug Administration)를 비롯한 대부분의 규제기관에서는 NCA하는 소프트웨어를 규정하고 있지 않아*, 상용 소프트웨어를 사용하지 않고 약동학적 지표를 구하는 것을 허용하고 있습니다.
따라서 무료로 누구나 사용할 수 있는 R 패키지를 사용하여 비구획분석을 통한 약동학적 주요 지표를 구할 수 있습니다.

- NonCompart [@R-NonCompart]
- ncar [@R-ncar]
- pkr [@R-pkr]

## 설치

우선 R을 설치합니다. 
R은 아래 링크^[https://cran.r-project.org/]에서 다운로드 받을 수 있습니다. 

R을 실행한 후, 콘솔 창에서 비구획분석을 위한 패키지를 설치하는 방법은 다음과 같습니다. 
홑따옴표 등의 인용 부호에 주의하세요.

```{r eval = FALSE}
install.packages('NonCompart')
install.packages('ncar')
install.packages('pkr')
```

설치는 한번만 하면 되지만, 비구획분석을 위해서는 매 세션마다 패키지를 *불러오기*해야 합니다.

```{r}
library(NonCompart)
library(ncar)
library(pkr)
```

아래 두 패키지는 비구획분석과는 관계없지만 자료 처리 혹은 그림 등을 그리는데 도움을 줍니다. 

```{r}
# install.packages(c('tidyverse', 'knitr')) # 설치 안되어 있으면 맨앞의 #을 지우고 설치.
library(tidyverse) # For presentation only, dplyr, tidyr, ggplot2
library(knitr) # For reports
```

도움이 필요할때는 맨 앞에 물음표를 붙여서 콘솔창에 입력하거나 `help()` 함수를 사용합니다.

```{r, eval = FALSE}
?NonCompart
help(tblNCA)
```

## R에 대하여

R [@R-base]은 통계 소프트웨어 입니다. 
굉장히 유용한 소프트웨어이지만 이에 대해 여기서 자세히 설명하긴 힘듭니다. 
R에 대한 많은 책들을 bookdown.org^[https://bookdown.org]에서 무료로 읽을 수 있습니다. 
Coursera^[https://coursera.com]에서 무료 온라인 강의를 들을 수 있습니다.

## 자료 탐색

자료의 첫 10개 (Table \@ref(tab:head)) 혹은 마지막 10개 관찰값만 보고 싶으면 다음을 입력합니다. 
대상자 번호가 첫 열에 나와있고 시간 순서대로 혈장에서 측정한 테오필린의 농도가 나와있습니다. 

```r
head(Theoph, n=10)
tail(Theoph, n=10)
```

```{r head, echo = FALSE}
kable(head(Theoph, n=10), caption = 'Theoph 자료의 첫 10개 관찰값',
      row.names = FALSE, booktabs = TRUE)
```

그림을 그려서 대략적인 자료의 모습을 파악합니다. (Figure \@ref(fig:ggtheoph))

```{r ggtheoph, fig.cap = 'Concentration-time curves of oral administration of Theoph (N = 12)', fig.width = 6, fig.height = 3.5}
ggplot(Theoph, aes(Time, conc, group = Subject, color = Subject)) +
  geom_point(size = 4) + 
  geom_line(size = 1) +
  theme_bw() +
  labs(title = 'Oral Administration of Theoph (320 mg)',
       x = 'Time (hour)', y = 'Concentration (ng/mL)')
```

## 파라메터의 의미

비구획분석 시 여러 파라메터가 나오며 약어로 표현하는 경우가 많습니다. 또한 소프트웨어마다 약어가 상이하기 때문에 자주 그 의미를 찾아볼 필요가 있습니다. 콘솔창에 다음을 입력합니다.

```{r eval = FALSE}
?ncar::txtNCA()
ncar::RptCfg
```

ncar::RptCfg의 일부를 첨부합니다. (Table \@ref(tab:rptcfg)) `PPTESTCD`는 NonCompart 패키지에서 출력하는 파라메터 이름이며, CDISC SDTM PPTESTCD (Parameter Short Name)^[다음과 같이 CDISC note에 표시되어 있습니다. 'Short name of the pharmacokinetic parameter. It can be used as a column name when converting a dataset from a vertical to a horizontal format. The value in PPTESTCD cannot be longer than 8 characters, nor can it start with a number (e.g., "1TEST"). PPTESTCD cannot contain characters other than letters, numbers, or underscores. Examples: "AUCALL", "TMAX", "CMAX".' https://wiki.cdisc.org/pages/viewpage.action?pageId=42309513]와 같은 값입니다. `WNL` 열은 Certara Phoenix WinNonLin에서 구한 파라메터 이름입니다.

```{r rptcfg, echo = FALSE}
kable(ncar::RptCfg %>% select(PPTESTCD, SYNONYM, WNL), caption = 'Description of NonCompart parameters',booktabs=TRUE, longtable = TRUE)
```

# 패키지: NonCompart

## sNCA()

한명의 대상자에 대해 비구획 분석을 시행합니다.

```{r}
# For one subject
x = Theoph[Theoph$Subject=="1","Time"]
y = Theoph[Theoph$Subject=="1","conc"]

sNCA(x, y, dose=320, doseUnit="mg", concUnit="mg/L", timeUnit="h")
```

이때의 그림은 다음과 같습니다.  (Figure \@ref(fig:ggtheophindi))

```{r ggtheophindi, fig.cap = 'Individual concentration-time curves of oral administration of Theoph (Subject 1)', fig.width = 6, fig.height = 3.5}
ggplot(Theoph %>% dplyr::filter(Subject == 1), 
       aes(Time, conc, group = Subject, color = Subject)) +
  geom_point(size = 4) + geom_line(size = 1) +
  theme_minimal() +
  labs(title = 'Oral Administration of Theoph (320 mg) (Subject 1)',
       x = 'Time (hour)', y = 'Concentration (ng/mL)')
```

## tblNCA(): 전체 대상자 비구획 분석

가장 많이 쓰는 함수 입니다! 
NonCompart 패키지의 핵심적인 기능입니다.
아래의 코드를 R의 콘솔창에 넣어보세요. 
테오필린 경구 투여시의 비구획 분석입니다. 

```{r}
Theoph_tblNCA <- tblNCA(Theoph, key="Subject", dose=320, concUnit="mg/L")
```

결과는 matrix 형태인데 너무 길기 때문에 핵심적인 일부 파라메터 (C~max~, T~max~, AUC~last~)만 표시할 수도 있습니다.

```{r}
Theoph_tblNCA_selected <- Theoph_tblNCA[ , c('Subject', 'CMAX', 'TMAX', 'AUCLST')]
Theoph_tblNCA_selected
```

인도메타신 정맥 투여시의 비구획 분석입니다. 
함수인자 `adm`을 infusion으로 바꾼 것을 볼 수 있고 `dur`가 추가된 것을 볼 수 있습니다.

```{r}
Indometh_tblNCA <- tblNCA(Indometh, key="Subject", colTime="time", colConc="conc", dose=25, 
       adm="Infusion", dur=0.5, concUnit="mg/L")
```

역시 핵심적인 일부 파라메터 (C~max~, T~max~, AUC~last~)만 표시할 수도 있습니다.

```{r}
Indometh_tblNCA_selected <- Indometh_tblNCA[ , c('Subject', 'CMAX', 'TMAX', 'AUCLST')]
Indometh_tblNCA_selected
```

## 기술통계 (Descriptive statistics)

R에서는 필요에 따라서 자신만의 함수를 만들 수도 있습니다. 
아래 두줄을 실행하면 `desc_tblNCA()` 함수를 사용하여 기술통계량을 쉽게 구할 수 있습니다. (Table \@ref(tab:theodesc) and \@ref(tab:indodesc))

```{r}
desc_tblNCA <- function(tblNCA){as.data.frame(tblNCA) %>% 
    mutate_all(function(x) as.numeric(as.character(x))) %>% broom::tidy()}
```

```{r eval = FALSE}
desc_tblNCA(Theoph_tblNCA_selected)
desc_tblNCA(Indometh_tblNCA_selected)
```

```{r theodesc, echo = FALSE}
desc_tblNCA(Theoph_tblNCA_selected) %>% kable(booktabs = TRUE, caption = 'Descriptive statistics of selected PK parameters of Theoph oral administration')
```

```{r indodesc, echo = FALSE}
desc_tblNCA(Indometh_tblNCA_selected) %>% kable(booktabs = TRUE, caption = 'Descriptive statistics of selected PK parameters of Indometh IV infusion')
```

# 패키지: ncar

보고서를 만드는 패키지입니다. 현재 설정된 working directory에 결과 파일이 생성됩니다.

## txtNCA()

txtNCA()를 통해서 다음 결과를 얻을 수 있습니다.

```{r}
txtNCA(Theoph[Theoph$Subject=="1","Time"],
       Theoph[Theoph$Subject=="1","conc"], 
       dose=320, doseUnit="mg", concUnit="mg/L", timeUnit="h")
```

파일로 저장하려면 다음을 입력합니다.

```{r}
writeLines(txtNCA(Theoph[Theoph$Subject=="1","Time"],
                  Theoph[Theoph$Subject=="1","conc"], 
                  dose=320, doseUnit="mg", concUnit="mg/L",
                  timeUnit="h"), 
           'Output-ncar/txtNCA-Theoph.txt')
```

## pdfNCA()

pdfNCA()로 pdf로 결과를 볼 수 있습니다. 

```{r pdfNCA}
pdfNCA(fileName="Output-ncar/pdfNCA-Theoph.pdf", Theoph, colSubj="Subject", colTime="Time", 
       colConc="conc", dose=320, doseUnit="mg", timeUnit="h", concUnit="mg/L")
```

## rtfNCA()

마이크로소프트 워드에서 편집가능한 rtf파일을 만듭니다.

```{r eval = FALSE}
rtfNCA(fileName="rtfNCA-Theoph.rtf", Theoph, colSubj="Subject", colTime="Time", 
       colConc="conc", dose=320, doseUnit="mg", timeUnit="h", concUnit="mg/L")
```

# 패키지: pkr

## plotPK()

여러가지 기본적인 그림을 그려봅니다. Output 폴더 아래에 여러 파일이 생성됩니다.

```{r eval = TRUE}
pkr::plotPK(Theoph, "Subject", "Time", "conc", 
            unitTime = "hr", unitConc = "mg/L", dose = 320)
```



# 기타 사항
