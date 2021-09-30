<h1 align="center"> Analytical List Page | ABAP CDS | UI5 🖥️ </h1>

<div align="center">
  Create an Analytical List Page using ABAP CDS views and annotations.
</div>
<div align="center">
  <h3> 📝 
    <a href="https://www.linkedin.com/in/gonzalo-meana-balseiro-90a523188/">
      Contact Me
    </a>
  </h3>
    <h3> 💻  
    <a href="http://gonzalomb.com">
      Check my website
    </a>
  </h3>
</div>

## Starting 🚀
Analytical List Page is a powerful Fiori Element available since SAPUI5 innovation version 1.48, this template provides the ability to create an analytical dashboard with KPIs, charts, tables and also a drill-down navigation to a detail page.

You can find all the relevant information about Analytical List Pages in the [SAP Fiori Design Guideline](https://experience.sap.com/fiori-design-web/analytical-list-page/).

### Pre-requirements 📋

_Tools you need to be able to develop this application_

* **SAP Logon** 
* **SAP Cloud Connector (SCC)** 
* **SAP Cloud Platform (SCP)** or **SAP Business Technology Platform (BTP)**
* **Eclipse**
* **SAP Web IDE** or **SAP BAS** 

## Practical case ⚙️

_In this application we are going to develop both the back-end and the front-end part_

### Back-end 🔩
#### 1. ABAP CDS
* **DIMENSION:** Airline 
```abap
@AbapCatalog.sqlViewName: 'ZDIMEAIRLINE'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Airline'

@Analytics.dataCategory: #DIMENSION

define view Z_Dimension_Airline
  as select from scarr
{
      @ObjectModel.text.element: [ 'AirlineName' ]
  key carrid   as Airline,
  
      @Semantics.text: true
      carrname as AirlineName,
      
      @Semantics.currencyCode: true
      currcode as Currency
} 
```

* **DIMENSION:** Connection 
```abap
@AbapCatalog.sqlViewName: 'ZDIMECONNECT'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Connection'

@Analytics.dataCategory: #DIMENSION

@ObjectModel.representativeKey: 'FlightConnection'

define view Z_Dimension_Connection
  as select from spfli
  association [0..1] to Z_Dimension_Airline as _Airline on $projection.Airline = _Airline.Airline
{
      @ObjectModel.foreignKey.association: '_Airline'
  key carrid                    as Airline,

      @ObjectModel.text.element: [ 'Destination' ]
  key connid                    as FlightConnection,

      @Semantics.text: true
      concat(cityfrom,
        concat(' -> ', cityto)) as Destination,

      _Airline
} 
```

* **DIMENSION:** Customer
```abap
@AbapCatalog.sqlViewName: 'ZDIMECUSTOMER'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Customer'

@Analytics.dataCategory: #DIMENSION

define view Z_Dimension_Customer
  as select from scustom
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'CustomerName' ]
  key id      as Customer,

      @Semantics.text: true
      name    as CustomerName,

      @ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country as Country,

      @Semantics.address.city: true
      city    as City,
      
      _Country
} 
```
* **DIMENSION:** Travel Agency
```abap
@AbapCatalog.sqlViewName: 'ZDIMETRVAGENCY'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Travel Agency'

@Analytics.dataCategory: #DIMENSION

define view Z_Dimension_TravelAgency
  as select from stravelag
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'TravelAgencyName' ]
  key agencynum as TravelAgency,

      @Semantics.text: true
      name      as TravelAgencyName,

      @ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country   as Country,

      @Semantics.address.city: true
      city      as City,
      
      _Country
} 
```

* **CUBE:** Flight Bookings
```abap
@AbapCatalog.sqlViewName: 'ZCUBEFLIGHTBOOK'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Bookings'

@Analytics.dataCategory: #CUBE

define view Z_Cube_FlightBookings
  as select from sbook
  association [0..1] to I_CalendarDate           as _CalendarDate on  $projection.FlightDate = _CalendarDate.CalendarDate
  association [0..1] to Z_Dimension_Airline      as _Airline      on  $projection.Airline = _Airline.Airline
  association [0..1] to Z_Dimension_Connection   as _Connection   on  $projection.Airline          = _Connection.Airline
                                                                  and $projection.FlightConnection = _Connection.FlightConnection
  association [0..1] to Z_Dimension_Customer     as _Customer     on  $projection.Customer = _Customer.Customer
  association [0..1] to Z_Dimension_TravelAgency as _TravelAgency on  $projection.TravelAgency = _TravelAgency.TravelAgency
{
  /** DIMENSIONS **/

  @EndUserText.label: 'Airline'
  @ObjectModel.foreignKey.association: '_Airline'
  carrid                 as Airline,

  @EndUserText.label: 'Connection'
  @ObjectModel.foreignKey.association: '_Connection'
  connid                 as FlightConnection,

  @EndUserText.label: 'Flight Date'
  @ObjectModel.foreignKey.association: '_CalendarDate'
  fldate                 as FlightDate,

  @EndUserText.label: 'Book No.'
  bookid                 as BookNumber,

  @EndUserText.label: 'Customer'
  @ObjectModel.foreignKey.association: '_Customer'
  customid               as Customer,

  @EndUserText.label: 'Travel Agency'
  @ObjectModel.foreignKey.association: '_TravelAgency'
  agencynum              as TravelAgency,

  @EndUserText.label: 'Flight Year'
  _CalendarDate.CalendarYear,

  @EndUserText.label: 'Flight Month'
  _CalendarDate.CalendarMonth,

  @EndUserText.label: 'Customer Country'
  @ObjectModel.foreignKey.association: '_CustomerCountry'
  _Customer.Country      as CustomerCountry,

  @EndUserText.label: 'Customer City'
  _Customer.City         as CustomerCity,

  @EndUserText.label: 'Travel Agency Country'
  @ObjectModel.foreignKey.association: '_TravelAgencyCountry'
  _TravelAgency.Country  as TravelAgencyCountry,

  @EndUserText.label: 'Travel Agency Customer City'
  _TravelAgency.City     as TravelAgencyCity,

  /** MEASURES **/

  @EndUserText.label: 'Total of Bookings'
  @DefaultAggregation: #SUM
  1                      as TotalOfBookings,

  @EndUserText.label: 'Weight of Luggage'
  @DefaultAggregation: #SUM
  @Semantics.quantity.unitOfMeasure: 'WeightUOM'
  luggweight             as WeightOfLuggage,

  @EndUserText.label: 'Weight Unit'
  @Semantics.unitOfMeasure: true
  wunit                  as WeightUOM,

  @EndUserText.label: 'Booking Price'
  @DefaultAggregation: #SUM
  @Semantics.amount.currencyCode: 'Currency'
  forcuram               as BookingPrice,

  @EndUserText.label: 'Currency'
  @Semantics.currencyCode: true
  forcurkey              as Currency,

  // Associations
  _Airline,
  _CalendarDate,
  _CalendarDate._CalendarMonth,
  _CalendarDate._CalendarYear,
  _Connection,
  _Customer,
  _Customer._Country     as _CustomerCountry,
  _TravelAgency,
  _TravelAgency._Country as _TravelAgencyCountry
} 
```
* **QUERY:** Flight Bookings
```abap
@AbapCatalog.sqlViewName: 'ZQUERYFLIGHTBOOK'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Bookings'

@Analytics.query: true
@VDM.viewType: #CONSUMPTION

define view Z_Query_FlightBookings
  as select from Z_Cube_FlightBookings
{
    /** DIMENSIONS **/
    
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Airline, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    FlightConnection, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    FlightDate, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Customer, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgency, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CalendarYear,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CalendarMonth,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CustomerCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CustomerCity,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCity,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    Currency,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    WeightUOM,
    
    /** MEASURES **/
    
    TotalOfBookings, 
    WeightOfLuggage,
    BookingPrice,
    
    @EndUserText.label: 'Average Weight Per Flight'
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationBehavior: #AVG
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationElements: [ 'Airline', 'FlightConnection', 'FlightDate' ]
    @AnalyticsDetails.query.formula: '$projection.WeightOfLuggage'
    @AnalyticsDetails.query.decimals: 0
    0 as AverageWeightPerFlight
} 
```

### Front-End ⌨️
#### 2. UI5 Fiori elements applications (Web IDE)
* Select Analytical List Page as the template.

![image](https://user-images.githubusercontent.com/55688528/135479408-7c20618b-c1f1-463f-ad13-37c837b30134.png)

* Fill the project name, title, namespace and description.

![image](https://user-images.githubusercontent.com/55688528/135481981-a1e2378e-ab95-42ec-b8c6-be770d3bf2da.png)

* Select the OData service Z_QUERY_FLIGHT_ALP_CDS.

![image](https://user-images.githubusercontent.com/55688528/135480190-62522ffd-56ab-475d-bf71-a63f89c0a1ac.png)

* Select the remote annotation file to expose the annotations generated through the ABAP CDS view, this file contains the XML output demonstrated in the previous section.

![image](https://user-images.githubusercontent.com/55688528/135480261-4b0fbd0b-7b0b-4da2-98d9-18ace545338e.png)

* Define the template configuration:

  ** OData Collection: Z_QUERY_FLIGHT_ALP
  ** Qualifier: Default
  ** Table Type: Responsive
  ** Auto Hide: TRUE

![image](https://user-images.githubusercontent.com/55688528/135480534-62b0240a-9737-47a4-b7ff-3f0b1022d281.png)

* Press Finish to conclude the Web IDE wizard. This is the structure of your project after you conclude all the steps.

![image](https://user-images.githubusercontent.com/55688528/135481001-94845e51-41d8-4f44-9ae1-529203abc7b1.png)

Inside the manifest.json we can find the code generated automatically based on our previous configuration through the Web IDE wizard.

 

KPI (manifest.json)
Place this snippet of code inside the keyPerformanceIndicators attribute:

  "keyPerformanceIndicators": {
      "WeightByCountry": {
          "model": "kpi",
          "entitySet": "Z_QUERY_FLIGHT_ALP",
          "qualifier": "KPIWeightByCountry"
      }
  }
Don’t forget to create a new model called kpi pointing to your data souce, in my example the model references the mainService data source but you could use a different source if you want.
* KPI (manifest.json)

```json
 "keyPerformanceIndicators": {
      "WeightByCountry": {
          "model": "kpi",
          "entitySet": "ZQUERYFLIGHTALP",
          "qualifier": "KPIWeightByCountry"
      }
  }
```
*Create a new model called kpi pointing to your data souce

![image](https://user-images.githubusercontent.com/55688528/135483043-7e55da2f-0cd3-4602-b3b2-716dfb15d554.png)

* Visual Filter (annotation.xml)

![image](https://user-images.githubusercontent.com/55688528/135483119-dbbe37e4-c07e-4e4c-b1d8-89498b262fb3.png)
![image](https://user-images.githubusercontent.com/55688528/135483179-286398b6-44df-4117-9435-b6009e06d13e.png)

This is the code generated :

```xml
<Annotations Target="Z_QUERY_FLIGHT_ALP_CDS.Z_QUERY_FLIGHT_ALPType/CalendarYear">
    <Annotation Term="Common.ValueList">
        <Record Type="Common.ValueListType">
            <PropertyValue Property="CollectionPath" String="Z_QUERY_FLIGHT_ALP"/>
            <PropertyValue Property="Parameters"/>
            <PropertyValue Property="PresentationVariantQualifier" String="FilterBookingsByYear"/>
        </Record>
    </Annotation>
</Annotations>
```
* Object Page (annotation.xml)

![image](https://user-images.githubusercontent.com/55688528/135483406-6c55ccde-b617-4fb5-b2bd-1df8fba4d11d.png)

This is the code generated :

```xml
<Annotation Term="UI.Facets">
    <Collection>
        <Record Type="UI.CollectionFacet">
            <PropertyValue Property="ID" String="MainSection"/>
            <PropertyValue Property="Label" String="{@i18n&gt;DETAILS}"/>
            <PropertyValue Property="Facets">
                <Collection>
                    <Record Type="UI.ReferenceFacet">
                        <PropertyValue Property="Target" AnnotationPath="@UI.LineItem"/>
                    </Record>
                </Collection>
            </PropertyValue>
        </Record>
    </Collection>
</Annotation>
```

