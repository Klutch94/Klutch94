- 👋 Hi, I’m @Klutch94
- This is my page where i store Rstudio and SAS code to help my work become easier, all of these code-snippets are either collected from myself or other examples all over the internet.

-------------------------------------------------------------------------SAS CODE-------------------------------------------------------------------------

**************************************Simples investigations and PROC univariate**************************************

*Printing out the distribution of a specific variable name within a certain class.
```
proc means data=alldat maxdec=2 nolabels missing n nmiss mean std;
    var age;
    class exposure;
run;
```
*Summaries a specific column in a data-sheet.
```
proc summary data=alldat;
    var nbirths;
    output out=births sum=;
run;
proc print data=births noobs; run; 
```
*Descreptive statistics of different variables and groups within data-set.
```
TITLE "Guido's Guide to PROC Univariate, NESUG 2009, Burlington, VT";
PROC UNIVARIATE DATA=Mylib.Choljg;
TITLE2 "Analysis of Age grouped by Sex";
 CLASS Sex;
 VAR Age;
RUN; 
```
*Descreptive statistics of different variables sorted by specific group within data-set with boxplot
```
PROC SORT DATA=Mylib.Choljg OUT=Choljg;
 BY Sex;
PROC UNIVARIATE DATA=Choljg PLOT;
TITLE2 "Analysis of Age by Sex";
TITLE3 "Plot Statement added for side by side Box Plot";
 BY Sex;
 VAR Age;
 RUN;
 ```
 *Descreptive statistics of specific variabel within data-set
 /*Other Selections: Moments, TestsforLocation, Quantiles, ExtremeObs 
 ```
LIBNAME Mylib 'C:\Users\GVSUG\Desktop\Joseph Guido';
ODS SELECT BASICMEASURES;
TITLE "Guido's Guide to PROC Univariate, NESUG 2009, Burlington, VT";
PROC UNIVARIATE Data=Mylib.Choljg;
TITLE2 "Basic Measures selected";
 VAR Chol1;
RUN;
```

*Is there a significant difference in the variable Cholesterol between site 1 and site 2?
```
LIBNAME Mylib 'C:\Users\GVSUG\Desktop\Joseph Guido';
ODS SELECT TESTSFORLOCATION;
TITLE "Guido's Guide to PROC Univariate, NESUG 2009, Burlington, VT";
PROC UNIVARIATE Data=Mylib.Choljg;
TITLE2 "Is there a statistically significant difference in Cholesterol";
TITLE3 "between Site 1 and Site 2?";
VAR Choldiff;
RUN; 
```
Select the 10 lowest and 10 highest values of Cholesterol Difference 
```
LIBNAME Mylib 'C:\Users\GVSUG\Desktop\Joseph Guido';
ODS SELECT EXTREMEVALUES;
TITLE "Guido's Guide to PROC Univariate, NESUG 2009, Burlington, VT";
PROC UNIVARIATE Data=Mylib.Choljg NEXTRVAL=10;
TITLE2 "10 lowest and 10 highest values of Cholesterol Difference";
VAR Choldiff;
RUN; 
```
Using the HISTOGRAM statement in PROC UNIVARIATE 
```
LIBNAME Mylib 'C:\Users\GVSUG\Desktop\Joseph Guido';
TITLE "Guido's Guide to PROC Univariate, NESUG 2009, Burlington";
PROC UNIVARIATE Data=Mylib.Choljg NOPRINT;
CLASS Site Sex;
TITLE2 "Using the HISTOGRAM statement in PROC UNIVARIATE";
TITLE3 "Grouped by Site and Sex";
VAR Choldiff;
HISTOGRAM / NORMAL (COLOR=RED W=5) NROWS=2 NCOLS=2;
RUN; 
```
**************************************Investigation of certain observations**************************************
* print all observations satisfying certain criteria, in this example "above 25" 
```
proc print data=alldat;
    where bmi > 25;
    var name gender smoking;
run;
```
* print the first 20 observations 
```
proc print data=alldat(obs=20);
run;
```
* print the observations 50 through 70 
```
proc print data=alldat(firstobs=50 obs=70);
run;
```
**************************************Import/export**************************************
* read from csv file separated by commas ;
```
proc import datafile="/path/to/data.csv" out=alldat dbms=csv replace;
    getnames=yes;
run;
```
* read from csv file separated by other separator ;
```
proc import datafile="/path/to/data.csv" out=alldat dbms=dlm replace;
    getnames=yes;
    delimiter=";"; /* use delimiter="|" for pipe separated files */
    guessingrows=max;
run;
```

**************************************Renaming or removing data**************************************
* rename a dataset ;
```
proc datasets nolist;
    change myoldname = newname;
quit; run;
```
* delete datasets by enumeration or by common prefix (here _tmp_) ;
```
proc datasets nolist nowarn nodetails;
    delete olddata _tmp_: ;
quit; run;
```
**************************************Graphs and figues**************************************
* preamble 
```
ods listing style=statistical gpath="/path/to/my/folder"; 
ods graphics on / reset=all imagename="my_filename" height=720px;
```

* histogram 
```
proc univariate data=alldat;
    histogram height / normal; * normal causes a normal distribution to be plotted ;
run;
```
* cumulative density function ;
```
proc univariate data=alldat;
    cdfplot height / normal noecdf; * noecdf leaves only the normal density function ;
run;
```
* barchart (grouped) where you want to hide a fake observation that has obsweight=0 ;
```
proc sgplot data=&dataset. pctlevel=group;
    vbar year / group=subtype stat=percent legendlabel="" name="a" legendlabel="" weight=obsweight;
    keylegend "a";
run;
```

**************************************Proc SQL**************************************

* summing/grouping over all age groups ;
```
proc sql noprint;
	create table autpop as 
		select year, gender, sum(population) as population
		from austrian_population 
		group by year, gender
		order by year, gender;
quit; 
run;
```
* left joining ;
```
proc sql noprint;
    create table alldat as
        select * 
        from rate_by_agegroup e left join population_by_agegroup a 
            on e.gender=a.gender and e.year=a.year and e.ag=a.ag;
quit;
run;
```
**************************************Sorting**************************************
* out= option is optional ;
* here: sort by lighest to heaviest ppl and oldest to youngest ppl ;
```
proc sort data=alldat out=alldat_sorted;
    by weight descending age;
run;
```
* keep only the first id, that is in the physical file ;
```
proc sort data=alldat nodupkey;
    by id;
run;
```
**************************************Creating quartiles**************************************
* out= option is important ;
```
proc rank data=alldat out=alldat groups=4;
    var bmi;
    ranks bmi_q;
run;
```
**************************************Missing obs**************************************
```
proc mi data=alldat nimpute=0;
    var age height bmi;
    ods select misspattern;
run;
```
**************************************Transposing**************************************
* prefix= and id are optional, cause the columns to have names ;
```
proc sort data=pop; by agegrp gender; run;

proc transpose data=pop out=lexis_pop prefix=period_;
    id period;
    var population;
    by agegrp gender;
run;
```


----------------------------------------------------------------------RSTUDIO CODE----------------------------------------------------------------------


<!---
Klutch94/Klutch94 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
