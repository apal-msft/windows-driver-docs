---
title: ServiceCategoryList
description: ServiceCategoryList
ms.date: 04/20/2017
---

# ServiceCategoryList

[!include[MBAE deprecation warning](../includes/mbae-deprecation-warning.md)]

The ServiceCategoryList element specifies one or more functional categories that apply to the service. Each functional category is specified through a [ServiceCategory](servicecategory.md) element.

## Usage


``` syntax
<ServiceCategorylist>
  child element
</ServiceCategorylist>
```

## Attributes


There are no attributes.

## Text value


Must contain one [ServiceCategory](servicecategory.md) element.

## Child elements


<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th>Element</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><a href="servicecategory.md" data-raw-source="[ServiceCategory](servicecategory.md)">ServiceCategory</a></p></td>
<td><p>Specifies one or more functional categories that apply to the service.</p></td>
</tr>
</tbody>
</table>

 

## Parent elements


<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th>Element</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><a href="serviceinfo.md" data-raw-source="[ServiceInfo](serviceinfo.md)">ServiceInfo</a></p></td>
<td><p>The <a href="serviceinfo.md" data-raw-source="[ServiceInfo](serviceinfo.md)">ServiceInfo</a> element is the parent element of the <a href="serviceinfo-xml-schema.md" data-raw-source="[ServiceInfo XML schema](serviceinfo-xml-schema.md)">ServiceInfo XML schema</a>.</p></td>
</tr>
</tbody>
</table>

 

## XSD


``` syntax
<xs:element name="ServiceCategoryList" type="tns:ServiceCategoryListType" />

<xs:complexType name="ServiceCategoryListType">
  <xs:sequence>
    <xs:element name="ServiceCategory" type="tns:ServiceCategoryType" maxOccurs="unbounded" />
    <xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded" />
  </xs:sequence>
</xs:complexType>
```

## Remarks


The following discusses the use of the ServiceCategoryList elements in a service metadata package:

-   The first [ServiceCategory](servicecategory.md) element in the ServiceCategoryList element specifies the service’s primary functional category. The primary functional category should match how the service is advertised, packaged, sold, and ultimately identified by users.

-   Because a service is defined only by its primary functional category, you should specify only one instance of the [ServiceCategory](servicecategory.md) element in the ServiceCategoryList element.

-   The [ServiceCategory](servicecategory.md) for service metadata packages must be one of the following:

    -   Network.MobileBroadband

    -   Network.MobileBroadband.CDMA

    -   Network.MobileBroadband.GSM

The ServiceCategoryList element is required.

 

 





