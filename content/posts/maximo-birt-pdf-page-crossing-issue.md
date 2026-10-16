---
title: "MAXIMO BIRT Report PDF format page crossing issue"
date: 2022-06-20
tags: [maximo, birt, reporting, troubleshooting]
draft: false
---


Yes, I am still working on BIRT report as fire fighter for one of my projects. Now was into another tricky issue. The BIRT report I am helping on looks all good except when try to export it to PDF. Some particular page the content cross the footer, such as below:

![](https://cdn-images-1.medium.com/max/800/1*h_P98mlv2b7JfkiTzK4tRQ.png)

Tried different options, such as setting page break, modified padding of the row. But none of them solved this issue. Tried asked MR. Google, but didn’t find any useful solution on it. The client can accept to move the paging number to the top, but it will require to update the master page template, and I thought it is not a good solution as well. I can understand this should be caused by pdf export function on calculating the page content might be different with how BIRT report, but cannot figure out which setting might cause the problem.

Took a deep breath, and walked away from computer, came back with refresh eyes and reviewed page setting from the below IBM website.

The following page setting caused my attention. There is slight difference between US letter and A4 size. In US, Letter is US version of A4 page size. The following image shows what’s the different of them.

![](https://cdn-images-1.medium.com/max/800/1*7bDn3Fdpf_2tGQBe_go17g.png)

When BIRT report export to PDF version, it might default to US letter page size. I went back to eclipse, opened the report, and located the master page size setting, the page type is A4, instead of US Letter.

![](https://cdn-images-1.medium.com/max/800/1*V-VgMYvkGfDyTfQlxu5Tew.png)

After change this page type to be US Letter, run the report again, the issue of content crossing the footer was gone. Tested different records in the report to make sure this issue is really fixed. Now can clapped my hand and release it to testing environment for business team to validate it.

Hope this article can give you a hint when you have the similar issue on BIRT report export to pdf.

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/maximo-birt-report-pdf-format-page-crossing-issue-dc590b4d5772).*
