# utl_five_algorithms_for_tranposing_multiple_pairs_of_variables
Five algorithms for tranposing multiple pairs of variables. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

StackOverflow R: Five algorithms for tranposing multiple pairs of variables

  All these solutions create tables, I am not interested in the report from proc report, just the output dataset.
  I think time spent with pro report is well spent.
  Especially if SAS decides to fix the ODS variable naming bug.
  proc corresp does honor ods output.

  Five solutions

     1. WPS/SAS Just 'proc report'
     2. Gather and proc corr  (don't have gather in WPS yet)
     3. WPS/SAS Two transposes
     4. WPS/Proc R
         a. reshape2 dcast melt
         b. data.table dcast

github
https://tinyurl.com/yauwa4yc
https://github.com/rogerjdeangelis/utl_five_algorithms_for_tranposing_multiple_pairs_of_variables

stackoverflow R
https://stackoverflow.com/questions/50449604/transform-a-data-frame

Rui Barradas profile
https://stackoverflow.com/users/8245406/rui-barradas

SCDCE profile
https://stackoverflow.com/users/4828653/scdce

INPUT
=====

SD1.HAVE total obs=4

  Month    Currency    value1    value2

   Jan      euro         210       200
   Jan      dollar       120       300
   Feb      euro         100       280
   Feb      dollar       200       150

EXAMPLE OUTPUT

         value1   value1     value2    value2
 Month    euro    dollar     euro      dollar
  Jan     210      120       200        300
  Feb     100      200       280        150


WORK.HAVCOR total obs=3

           value1___    value1___    value2___    value2___
  Label      dollar        euro        dollar        euro       Sum

   Feb        200          100          150          280        730
   Jan        120          210          300          200        830

   Sum        320          310          450          480       1560

WORK.HAVXPOXPO total obs=2

 Month    _NAME_    value1euro    value2euro    value1dollar    value2dollar

  Jan      COL1         210           200            120             300
  Feb      COL1         100           280            200             150

PROCESS
=======

 1. WPS/SAS Just proc report
    (unfortunately ods proc report does not honor the report)

    proc report data=sd1.have missing nowd
       out=wantrpt(rename=( /* variables come out in alphabetic order */
         _c2_ = value1_dollar
         _c3_ = value1_euro
         _c4_ = value2_dollar
         _c5_ = value2_euro
      )) ;
    col month currency, (value1 value2);
    define month / group;
    define currency / across;
    define value1 /  sum;
    define value2 /  sum;
    run;quit;

 2. Gather and proc corresp  (don't have gather in WPS yet)

    %utl_gather(sd1.have,var,val,month currency,havGat,valformat=5.);

    ods exclude all;
    ods output observed=havCor;
    proc corresp data=havGat observed dim=1 cross=both;
      tables month, var currency;
      weight val;
    run;quit;
    ods select all;

 3. WPS/SAS Two transposes

    proc transpose data=sd1.have out=havXpo;
      by month currency notsorted;
      var value1 value2;
    run;quit;

    proc transpose data=havXpo out=havXpoXpo;
      by month notsorted;
      id  _name_ currency;
      var col1;
    run;quit;

 4. WPS/Proc R (working code)

    Reshape
      want_reshape<-dcast(melt(have, id.vars = c("MONTH", "CURRENCY")), MONTH ~ CURRENCY + variable);

    data.table
      want_data_table<-dcast(setDT(have), MONTH ~ CURRENCY, value.var = c(rep(paste0("VALUE", 1:2))));


OUTPUT
======

1. WPS/SAS Just proc report

          value1             value2
  Month    euro    dollar     euro      dollar
   Jan     210      120       200        300
   Feb     100      200       280        150


2. Gather and proc corresp  (don't have gather in WPS yet)

 WORK.HAVCOR total obs=3

           value1___    value1___    value2___    value2___
  Label      dollar        euro        dollar        euro       Sum

   Feb        200          100          150          280        730
   Jan        120          210          300          200        830

   Sum        320          310          450          480       1560

3. WPS/SAS Two transposes

 WORK.HAVXPOXPO total obs=2

 Month    _NAME_    value1euro    value2euro    value1dollar    value2dollar

  Jan      COL1         210           200            120             300
  Feb      COL1         100           280            200             150

4. WPS/PROC R RESHAPE

 WANT_RESHAPE total obs=2

           DOLLAR_    DOLLAR_     EURO_     EURO_
  MONTH     VALUE1     VALUE2    VALUE1    VALUE2

   Feb       200        150        100       280
   Jan       120        300        210       200

5. WPS/PROC R DATA_TABLE

 WANT_DATA_TABLE total obs=2

           VALUE1_    VALUE1_    VALUE2_    VALUE2_
  MONTH     DOLLAR      EURO      DOLLAR      EURO

   Feb       200        100        150        280
   Jan       120        210        300        200

*                _               _       _
 _ __ ___   __ _| | _____     __| | __ _| |_ __ _
| '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
| | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
|_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

;

options validvarname=upcase;
libname sd1 "d:/sd1";
data sd1.have;
 input Month $ Currency $ value1 value2;
cards4;
 Jan euro 210 200
 Jan dollar 120 300
 Feb euro 100 280
 Feb dollar 200 150
;;;;
run;quit;

*                               _       _   _
__      ___ __  ___   ___  ___ | |_   _| |_(_) ___  _ __  ___
\ \ /\ / / '_ \/ __| / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
 \ V  V /| |_) \__ \ \__ \ (_) | | |_| | |_| | (_) | | | \__ \
  \_/\_/ | .__/|___/ |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/
         |_|
;

*WPS PROC REPORT;
%utl_submit_wps64('
libname sd1 sas7bdat "d:/sd1";
libname wrk sas7bdat "%sysfunc(pathname(work))";
proc report data=sd1.have missing nowd
   out=wrk.wantRptWps(rename=( /* variables comme out in alphabetic order */
     _c2_ = value1_dollar
     _c3_ = value1_euro
     _c4_ = value2_dollar
     _c5_ = value2_euro
  )) ;
col month currency, (value1 value2);
define month / group;
define currency / across;
define value1 /  sum;
define value2 /  sum;
run;quit;
');

* WPS two transposes;

%utl_submit_wps64('
libname sd1 sas7bdat "d:/sd1";
libname wrk sas7bdat "%sysfunc(pathname(work))";

proc transpose data=sd1.have out=havXpo;
by month currency notsorted;
var value1 value2;
run;quit;

proc transpose data=havXpo out=wrk.havXpoXpo;
by month notsorted;
id  _name_ currency;
var col1;
run;quit;
');

* do nat have gather working in WPS yet;
%utl_gather(sd1.have,var,val,month currency,havGat,valformat=5.);

ods exclude all;
ods output observed=havCor;
proc corresp data=havGat observed dim=1 cross=both;
tables month, var currency;
weight val;
run;quit;
ods select all;

* WPS PROC R;

%utl_submit_wps64('
options set=R_HOME "C:/Program Files/R/R-3.3.2";
libname wrk  sas7bdat "%sysfunc(pathname(work))";
options validvarname=upcase;
proc r;
submit;
source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
library(haven);
library(reshape2);
library(data.table);
have<-read_sas("d:/sd1/have.sas7bdat");
head(have);
want_reshape<-dcast(melt(have, id.vars = c("MONTH", "CURRENCY")), MONTH ~ CURRENCY + variable);
want_data_table<-dcast(setDT(have), MONTH ~ CURRENCY, value.var = c(rep(paste0("VALUE", 1:2))));
endsubmit;
import r=want_reshape      data=wrk.want_reshape;
import r=want_data_table   data=wrk.want_data_table;
run;quit;
');


