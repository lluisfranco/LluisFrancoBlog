---
title: "The missing cube"
subtitle: "OLAP cube is not visible in Excel"
date: 2025-06-17T10:21:10+02:00
featuredImage: /images/posts/Post_AnalysisServicesLogo.png
fontawesome: "true"
categories: 
- SQL Server
tags:
- Analysis Services
- OLAP
draft: true
---
## The issue

When browsing analysis services cubes from Excel there's only one cube that doesn't appear. The tricky part it's the only one :S

After a little bit of research it seems the cube as been deployed with the visible attribute to false.

### Solution 1: Keep the cube invisible but accessible from Excel

In Excel you can change the connection string just adding the attribute 'Show Hidden Cubes=true;"

Open a blank workbook and go to the get data menu

{{< figure src="/images/posts/ExcelGetDataFromAS.png" title="Get Data From Analysis Services" >}}

Then connect to your Analysis Service and select another visible cube, we will change it later

{{< figure src="/images/posts/SelectCube.png" title="Selecting a cube" >}}

After connecting, let's change the connection string. Go to DATA tab / Existing Connections 

{{< figure src="/images/posts/EditASConnection.png" title="Editing connection 1" >}}

Right click the current book connection and edit connection properties

{{< figure src="/images/posts/EditCubeProperties.png" title="Editing connection 2" >}}

Then add the following attribute to the connection string, and replace the command (the name of the hidden cube)

**;Show hidden cubes=true**

> Note: This solution is also useful when connecting to Analysis Services using SQL Server Management Studio

### Solution 2: Change the attribute visible making the cube visible again

Open Sql Server Management Studio and connect to your Analysis Services, then navigate to the hidden cube (should be visible in under the Cubes node).
As you can see there is no way to change the Visible property. However it's possible to right click the cube and script cube as alter.
After a moment this will generate the cube script. Once created search for:

<Visible>false</Visible>

And replace it for:

<Visible>true</Visible>

> Beware! Change this property only for the cube, it may appear also in hidden dimensions.

Execute the script and process the cube ;)

Kind regards from Andorra,