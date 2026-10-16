---
title: "How to use Regular Expression in Maximo BIRT Report"
date: 2022-06-10
tags: [maximo, birt, reporting]
draft: false
---


If you read my [previous blog](https://jackyqiubao.medium.com/solution-for-maximo-birt-report-exception-of-mxreportscriptcontext-is-not-defined-320d36088f49), you might know I am recently working as fire fighter on Maximo BIRT reports. One of requirement on a complicated report is required to retrieve number inside parentheses from long description, for example, need to get (2) from the following long description content:

*(2) this is long description text, might content rich text format*

The solution in my mind when I heard this requirement is to use regular expression. Since in current project the Maximo database is DB2, asked Mr. Google to found DB2 did support regular expression in the query. After looked up regular expression functions on DB2 document, REGEXP_INSTR function is what I need. For example, if you run the following query:

*select REGEXP_Substr(‘(2) this is long description text, might content rich text format’,’\(\b[0–9]+\b\)’,1,1) from sysibm.sysdummy1*

You will get the result as:

(2)

![](https://cdn-images-1.medium.com/max/800/1*guvFpgH3F2M73-35KuCgOg.png)

You can refer to DB2 document on more detail on REGEXP_INSTR function:

After implemented this logic in the query, tested the whole query (more than 900 lines ), the result is what I expected. Merged the changes of the query to Maximo BIRT report script at the dataset open event. Ran the report in eclipse, the result was not what I expected, the REGEXP_INSTR function didn’t retrieve the number from the long description content. Went to the report log which contain the printed sql, found something interested in query string which was generated in the BIRT report script, the pattern expression string in REGEXP_INSTR function was changed to some characters:

![](https://cdn-images-1.medium.com/max/800/1*Qx9KGDGPJqD6i_uaf__v7g.png)

I guessed it might be caused by the backslash \ is the special character in the BIRT report script. It might require double backslash to get one backslash in the script. After made the following change, the result of the report worked as I expected.

![](https://cdn-images-1.medium.com/max/800/1*NEx40AU4_4-4oS1-NAkfNw.png)

The regression expression is really powerful tool for handling the string. If you are interested, you can also refer to DB2 Regular expression control characters document (other languages / systems have pretty similar on regular expression control characters)

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/how-to-use-regular-expression-in-maximo-birt-report-6ddd209fa69c).*
