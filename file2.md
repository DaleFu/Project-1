### H3

Alt-H1
======

Alt-H2
------



1. First ordered list item
2. Another item
⋅⋅* Unordered sub-list. 
1. Actual numbers don't matter, just that it's a number
⋅⋅1. Ordered sub-list
4. And another item.

⋅⋅⋅You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

⋅⋅⋅To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
⋅⋅⋅Note that this line is separate, but within the same paragraph.⋅⋅
⋅⋅⋅(This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

* Unordered list can use asterisks
- Or minuses
+ Or pluses



[I'm an inline-style link](https://www.google.com)


| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |



```SAS
/*This program creates the comp.fundaclean file which includes lagged variables and only observations where 
	indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C' 
	LAST RAN: 8/2/2018
*/
	/*Download comp.funda from wrds*/
		%let wrds = wrds.wharton.upenn.edu 4016;options comamid = TCP remote=WRDS;
		signon username=_prompt_;
		rsubmit;
			data fundaclean;
				set comp.funda; 
				where indfmt='INDL' and datafmt='STD' and popsrc='D' and consol='C';
			run;
			proc sort data=fundaclean; by gvkey datadate; run;
		*segment data;
			data segment;
				set compd.WRDS_SEGMERGED;
			    if stype IN ("BUSSEG") or stype IN ("OPSEG"); 
			    if soptp1 IN ("GEO") then delete;
			    if 2000 <= year(datadate) <= 2017;  
			    if datadate = srcdate; 
			    if sics1 ne "";
				if month(datadate)<=6 then fyear= year(datadate)-1; 
				if month(datadate)>6 then fyear= year(datadate);
				key = gvkey || "_" || fyear;
			run;
			proc sql;
				create table segment as
				select distinct gvkey, fyear, count(*) as segment, log(calculated segment + 1) as ln_segment
				from segment
				group by gvkey, fyear;
			quit;
			proc sql;
				create table fundaclean as
				select a.*, b.segment, b.ln_segment
				from fundaclean a left join segment b
				on a.gvkey=b.gvkey and a.fyear=b.fyear;
			quit;
			proc download data=fundaclean out=comp.fundaclean; run;
		endrsubmit;

```
