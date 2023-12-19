---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

![](../assets/img/C0323301-CERN_Computer_Centre.jpg){:width="60%"}
<p style="text-align: center;">Image credit: <a href="https://www.sciencephoto.com/media/800168/view/cern-computer-centre">CERN / Science Photo Library</a></p>

The previous lessons have shown how CMS Open Data MiniAOD files can be processed through tools like the Physics Object Extractor Tool to produce ROOT files with ``flat" structure `TTree` objects containing event and physics object information. You have also seen how such ROOT files can be analyzed using python tools like Coffea. 

On small scales, all of these techniques can be tested and applied to a specific analysis using docker resources on your local machine. But in order to process all of the Open Data needed for a typical physics search or measurement, more computational resources are a great help. 

Many distributed computing resources are accessible to Open Data researchers! Many universities and laboratories host Linux computing clusters to which you may have access. Cloud computing is also available to the general public for various fees through (at least) Google, Amazon, and Microsoft platforms. Thanks to the participation of **Tata Institute of Fundamental Research (TIFR)** in this workshop, this lesson will show you how to analyze full Open Data datasets using an **HTCondor** queue system on a Linux cluster. 

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> For this lesson you will log in to a computing cluster provided by the Tata Institute of
> Fundamental Research (TIFR). Open a terminal on your computer from which you can use `ssh`.
{: .prereq}

{% include links.md %}
