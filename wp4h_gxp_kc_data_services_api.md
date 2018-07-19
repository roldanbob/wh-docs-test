---

copyright:
  years: 2017, 2018
lastupdated: "2018-07-17"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Data services APIs

{{site.data.keyword.wh_prodname_long}} Data Services includes APIs for FHIR, data lake, and data reservoir that you can use to manage patient data from medical devices and other sources. Each service that is part of the {{site.data.keyword.wh_prodname_short}} Data Services has a set of APIs you can use to store, retrieve, and manage data. To help you understand how to use these APIs, Swagger documentation is provided, along with information about each API.

- [**FHIR API details**](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_details.html)<br/> {{site.data.keyword.wh_prodname_long}} supports a subset of the FHIR search parameters that are described in the FHIR DSTU2 specification. In addition, to use and understand the FHIR REST API, other information about the {{site.data.keyword.wh_prodname_short}} implementation is required.

**Parent topic:**

- [Health data services](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_data_services.html)

## FHIR Server API

FHIR resources are strictly formatted and designed to be shared between healthcare entities (such as between a patient's primary care physician, cardiologist, and a hospital system). FHIR resources come from many sources, including patient health records, medical devices (such as blood pressure monitors or scales), wearable fitness trackers, or other sources (such as weather data).

Related resources can be bundled together for ingestion, as described in the FHIR Specification under [Resource Bundle - Content ![External link icon](../../icons/launch-glyph.svg "External link icon")](hl7.org/fhir/dstu2/bundle.html){: new_window}.

The {{site.data.keyword.wh_prodname_short}} FHIR service API supports the resources that are described in [Table 1](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_service.html#wp4h_gxp_c_fhir_service__table_ckk_scb_hbb).

Note: Calling operations for unsupported FHIR resource types returns a 400 `Bad Request` error.

### FHIR API methods
{: #concept_isb_c13_1cb__section_dz1_tt1_tcb}

For any resource (or bundle of resources), you can call the following methods:

- Upload a new resource to the FHIR repository: `POST [base]/fhir-server/api/v1/{resource type}`
- Update an existing resource: `PUT [base]/fhir-server/api/v1/{resource type}/{logical id}`
- Search for information about a specific resource from the FHIR repository:`GET [base]/fhir-server/api/v1/{resource type}/{search parameters}` Note: {{site.data.keyword.wh_prodname_short}} supports a subset of the FHIR DSTU 2 search parameters. For more information about search parameters, see the [FHIR DSTU2 specification, Search ![External link icon](../../icons/launch-glyph.svg "External link icon")](http://hl7.org/fhir/DSTU2/search.html){: new_window} section.
- Download a specific resource: `GET [base]/fhir-server/api/v1/{resource type}/{logical id}`
- Download the history information for a resource: `GET [base]/fhir-server/api/v1/{resource type}/{logical id}/_history`
- Download a specified version of a resource: `GET [base]/fhir-server/api/v1/{resource type}/{logical id}/_history/{version id}`
- Remove a specified version of a resource: `DELETE [base]/fhir-server/api/v1/{resource type}/{logical id}` Note: The DELETE API performs a soft delete, that is, data is marked as deleted but not actually removed from the data store.

### API URL elements
{: #concept_isb_c13_1cb__section_n54_4xd_ycb}

The following URL elements can be used within the calls:

- `[base]` is the URI for the method, for example: `https://demo.watson-health.local/`
- `{resource type}` is one of the supported FHIR resource types or resource names that are shown in [Table 1](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_service.html#wp4h_gxp_c_fhir_service__table_ckk_scb_hbb). Note: In certain circumstances, you can specify a resourceName instead of a resourceType. For example, the Observation resourceType is associated with a number of different resourceNames, such as Observation (Device)::MedicalDeviceObservation and Observation (Patient):: ParticipantObservation (Height) . These resources are routed to different data stores; in this case, you can specify the MedicalDeviceObservation and ParticipantObservation resourceNames. The FHIR API associates the resourceName with the correct resourceType and routes the resources correctly.
- `{resource type properties}` are the extensions and properties that provide details about each resource type. The extensions are not explicitly called out, but these elements are also needed for POST and PUT methods.
- `{logical id}` is the UUID that is uniquely identifies the resource. The logical ID is generated when the resource is POSTed to the FHIR repository.
- `{version id}` is a UUID that uniquely identifies a specific version of a resource.

For more information about using the FHIR API, see [FHIR API details](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_details.html).

## Data Lake API

Use the {{site.data.keyword.prodname_dl}} REST API to upload and download non-FHIR (file) data directly to and from the data lake. {{site.data.keyword.prodname_dl}} APIs use certificate-based authentication and white-listing. API requests come from a client application that is running within the {{site.data.keyword.wh_prodname_short}} (rather than a person, as with the Data Reservoir API). The {{site.data.keyword.prodname_dl}} API takes the following path:

1. The client application uses a key and certificate to send a request to the {{site.data.keyword.prodname_dl}} API. Before you can use the {{site.data.keyword.prodname_dl}} API, you need to complete a few setup steps to make sure that you have a signed certificate:

    1. From your {{site.data.keyword.prodgroupname_short}} Consultant, request a key and certificate for each instance of your client application. The key and certificate files are signed and issued by the {{site.data.keyword.prodgroupname_short}} certificate authority. Each certificate file includes a CN (Common Name) and OU (Organization Unit) value. The CN value is unique for each instance of your client application. The OU is always {{site.data.keyword.prodgroupname_short}} (as the issuing organization).
    1. After you have the keys and certificates, work with your {{site.data.keyword.prodgroupname_short}} Consultant to open a ticket to onboard your CN information to the data lake service. After this step is complete, you can call the {{site.data.keyword.prodname_dl}} APIs to upload and download data.

    Note: The client certificate must be signed by the {{site.data.keyword.prodgroupname_short}} certificate authority or the request is rejected.

1. The {{site.data.keyword.prodname_dl}} Server uses an internal white-list to verify the CN (Common Name) and OU (Organization Unit) values of the client certificate. The verification helps ensure that a client with the specified CN and OU is authorized to make the request.
1. Additionally, calls to the {{site.data.keyword.prodname_dl}} API require the IBM-DP-correlationId and IBM-App-User headers from the client application, which are used as elements in audit records. The IBM-DP-correlationId specifies a unique ID to link audit records across the client application and the {{site.data.keyword.prodname_dl}} service. The IBM-App-User specifies the user information to be captured in the audit records.

Use the following methods to upload and download data to and from the data lake:

- Upload data into the data lake:`POST [base]/datalake/api/v1/data`
- Download data from the data lake:`GET [base]/datalake/api/v1/data/{file_id}`

For example, in the following cURL command, the client application uses the POST method to upload an HL7 file called `testdata.txt`.

```
curl -X POST \
   https://watson-health.example.com/datalake/api/v1/data \
    --key <client key> --cert <client cert> \
     -H "IBM-Filename: test_file" \
     -H "IBM-App-User: test_user" \
     -H "IBM-DP-correlationid: exsmple_correlationId" \
     -H "content-type: application/octet-stream"  \
     --data-binary "@testdata.txt"
```
{: screen}

When the file is uploaded, the {{site.data.keyword.prodname_dl}} service assigns a unique ID (UID), for example, `1bc5201d5c24401d8adf48a9fe7a86b6`. If you want to access or download this file, be sure to save the UID, along with the associated file name.

To download data that is already in the data lake, you need the UID that was received when you uploaded the data. For example, the following cURL command retrieves the file by the UID that was returned by the POST method.

```
curl -X GET \
    https://watson-health.example.com/datalake/api/v1/data/1bc5201d5c24401d8adf48a9fe7a86b6 \
     --key <client key> --cert <client cert> \
     -H "IBM-App-User: test_user" \
     -H "IBM-DP-correlationid: example_correlationId"
```
{: screen}

Note: FHIR data is moved to a separate FHIR data lake from the FHIR repository by the replicator, and copied from the FHIR data lake by the Personal Information Protection service. For more information, see the [Personal information protection service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AV5Iwg41SgsFwvqnf1Ku){: new_window}.

![Link to Data&#xA;Lake API Swagger.](userImages/swagger_button.JPG) {: #concept_z3g_lyh_ccb__imagemap_ajw_fp4_ccb2}

<map name="d6854e335" id="d6854e335">
<area href="gxp_datalake_ext_swagger.yaml" alt="" title="" shape="rect" coords="-1, 3, 122, 36" />
</map>

## Data Reservoir API

The data reservoir contains fully de-identified data that is ready to be used for analysis. Use the {{site.data.keyword.prodname_dr}} API for the following tasks:

- Upload file data into the data reservoir.
- Download file data from the data reservoir into external files.
- Search for and retrieve files by any of the following criteria:

    - `fromWhen`/`toWhen` - Specify a time period for which to search for records. `fromWhen` and `toWhen` specify a date and time to begin and end searching for records. Date must be in the following format: yyyy-MM-dd**T**hh:mm:ss**Z,**where time is in Coordinated Universal Time.
    - `userId` - Optional. Retrieves records that were uploaded by the specified data reservoir user.
    - `filename` - Optional. Retrieve a file by its name. Be sure to specify the name of the file that is stored in the data reservoir.
    - `maxRecords` - Optional. Specify the number of records to retrieve (up to 1000). If not specified, the API returns the first (that is, oldest) 1000 records in the specified time period.

    Note: An authorized user can download any file that was uploaded into the data reservoir via the {{site.data.keyword.prodname_dr}} API. Note: If you upload data directly into the data reservoir, make sure that the data does not contain any PHI, PII, or SPI. It is the study designer's role to ensure that all data that is uploaded directly by using the {{site.data.keyword.prodname_dr}} APIs is properly de-identified and anonymous.

Before you can use the {{site.data.keyword.prodname_dr}} API, you need a user ID and password that is managed by a third-party Identity Management service. For information about getting a user ID and password, talk to your {{site.data.keyword.prodgroupname_short}} Consultant. After you obtain your user ID and password, you can use them to invoke the {{site.data.keyword.prodname_dr}} APIs.

Use the following methods to upload and download data to and from the data reservoir:

- Upload data into the data reservoir:`POST [base]/datareservoir/api/v1/data`
- Download data from the data reservoir:`GET [base]/datareservoir/api/v1/data/{file_id}`

The following example uses a cURL command to add anonymized weather data to the data reservoir:

```
curl -X POST  \
    https://watson-health.example.com/datareservoir/api/v1/data \
      --data-binary "@study_1247_weather_data.csv" \
      --user <user id> \
      -H "IBM-Filename: weather_data" \
      -H "content-type: application/octet-stream"
```
{: screen}

Note: The user is prompted for a password when the API runs. When the file is uploaded, {{site.data.keyword.wh_prodname_short}} assigns a UID such as `w1-1bc5201d5c24401d8adf48a9fe7a5498`. If you want to access or download this file, you need the UID, which you can retrieve by using the metadata resource.

Note: You can validate that data is uploaded correctly in a number of different
ways, such as using a checksum:

1. Calculate a checksum before you upload the file.
1. When you upload the file, a checksum is returned in the response.
1. Compare the two checksums to validate that the data is uploaded correctly.

Data that is uploaded to the data reservoir can also be downloaded back to external files. For example, use the following command to download the CSV file that you previously uploaded from the data reservoir.

```
curl -X GET \
    https://watson-health.example.com/datareservoir/api/v1/data/w1-1bc5201d5c24401d8adf48a9fe7a5498\
      --user <user id> \
      --output c: \mydirectory\download_study_1247_data.csv
```
{: screen}

Note: Use the <code>--output (-o)</code> parameter to specify a download directory and file name as part of the request. If you do not specify a directory, then the downloaded data is saved to the directory from where the curl request was run.

Data can also be retrieved for a specified time period, for a specified user, or by file name. Use the following command to return the UID of the `study_1247_weather_data.csv` file that was uploaded to the data reservoir. You can then use the UID to download the file. If you don't know when the file was uploaded, use a long range for `fromWhen` and `toWhen`.

```
curl -X GET \
  https://watson-health.example.com/datareservoir/api/v1/metadata?fileName=study_1247_weather_data.csv?fromWhen=2010-01-01T00:00:00:Z&toWhen=2017-12-31T00:00:00Z \
     --user <user id>
```
{: screen}

The `fromWhen` and `toWhen` parameters are the date and time are the begin and end times to retrieve records. Date must be in the following format: yyyy-MM-dd**T**hh:mm:ss**Z,**where time is in Coordinated Universal Time. `fromWhen` and `toWhen` are required.

Use the following command to return all (up to the maximum of 1000) records that were uploaded by user 12345 during November 2017:

```
curl -X GET \
  https://watson-health.example.com/datareservoir/api/v1/metadata?userId=12345?fromWhen=2017-11-01T00:00:00:Z&toWhen=2017-12-01T00:00:00Z \
    --user <user id>
```
{: screen}

If the file data is in a format that can be read by a Spark job, user analysts can use Spark to add the data to Jupyter Notebook. From the file data reservoir, Spark has been tested with data in CSV, JSON, Parquet, and TXT formats. Other formats might work, but have not been tested.

Note: To use data from the data reservoir in a Spark job, you need the file name of the file in the data reservoir. Be sure to note the file name when you POST the file to the data reservoir.

For more information about using Spark jobs with data in the file data reservoir, see the {{site.data.keyword.wh_prodname_short}}: Analyst User Guide for the Analytics Subsystem.

![Link to Data&#xA;Reservoir API Swagger.](userImages/swagger_button.JPG) {: #concept_pbv_pyh_ccb__imagemap_tnk_jp4_ccb3}

<map name="d6854e542" id="d6854e542">
<area href="gxp_datareservoir_ext_swagger.yaml" alt="" title="" shape="rect" coords="-1, 3, 122, 36" />
</map>

## Medication Database API

{{site.data.keyword.wh_prodname_long}} provides an optional medication database that uses the contents of World Health Organization Drug Dictionary (WHO-DD). For access to the medication database, you must have a valid license from the [Uppsala Monitoring Centre ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://who-umc.org/){: new_window} (UMC) that covers intended use cases. If you do not have valid WHO-DD license, the medication database is still available, but it does not contain any WHO-DD content.

The medication database uses the FHIR medication resource API to access World Health Organization drug dictionary (WHO-DD). Queries made that use the FHIR medication resource API to access drug information can result in multiple versions of the drug information and multiple fields in the medication database.

- Download a specific medication: `GET [base]/meddb-server/api/v1/Medication/{logical id}`
- Download the history information for a medication: `GET [base]/meddb-server/api/v1/Medication/{logical id}/_history`
- Download a specified version of a medication: `GET [base]/meddb-server/api/v1/Medication/{logical id}/_history/{version id}`
- Search for information about a specific medication from the FHIR repository: `GET [base]/meddb-server/api/v1/Medication/{search parameters}`

For more information, see [Medication database](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_kc_medication_database.html).
