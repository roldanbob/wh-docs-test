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

 # Use cases Data Services are the heart of {{site.data.keyword.wh_prodname_long}}. Use Data Services to add your data to the {{site.data.keyword.wh_prodname_short}} , prepare it, and make it available for scientific analysis in the data mart or for other uses.

The {{site.data.keyword.wh_prodname_short}} is designed for flexibility. The following use cases can help you understand how you can use the {{site.data.keyword.wh_prodname_short}} data services to make your data available for detailed analysis.

Note: The examples in this section contain generic URL endpoints, user names, and other parameters. When you create your own APIs, be sure to replace any generic text with the correct parameters for your site.

## Study design: Creating an asthma inhaler study
{: #concept_d5b_djm_p1b__section_gfp_qpt_51b}

A set of clinicians and research scientists want to create a study that is based on inhaler use by patients with asthma. The inhaler for this study is a connected device that produces a rich set of information each time it is used, including (but not limited to) the following data:

- Patient ID
- Inhaler ID
- Inhaler dosage
- Inhaler percentage (did the patient get the full suggested dosage?).
- Time and date of use

In addition, patients for this study wear fitness trackers that provide the following information:

- Heart rate
- Blood pressure
- Daily exercise details
- Sleep records

The study designers might also want, or need, the following information:

- Practitioner ID (and other details).
- Patient details, such as address, age, and gender.
- Patient Consent data, which ensures that patients have given explicit permission to use their data for this study.
- Patient insurance and primary care physician information.
- Patient medical details, such as family medical and social history, co-morbidities, or age at asthma diagnosis.
- Patient lifestyle details, such as job, exercise history, dietary history, and substance abuse history
- Daily weather details, such as temperature, smog (particulate matter), and relative humidity.
- Data sets from previous studies.

Data for use in this study (or set of studies) can come from various sources, such as:

- FHIR resources from the patient's inhaler and fitness tracker, a patient questionnaire, and weather observations.
- HL7-formatted data from the patient's Electronic Health Record (EHR).
- Image data from CT scans or x-rays.
- Data in SAS data sets or CSV files that contains previously de-identified data from previous studies.

During the study design phase, designers work with {{site.data.keyword.deptname_whc}} , data scientists, and other personnel to determine what data is needed for a study. Then, depending on the data source, use the FHIR server REST API, the {{site.data.keyword.prodname_dl}} REST API, or the {{site.data.keyword.prodname_dr}} REST API to upload and manage the data.

## Use case 1: Adding FHIR data to the FHIR repository
{: #concept_d5b_djm_p1b__fhir_usecase}

After you design the study, you know which FHIR resources are needed. Use the FHIR Service APIs to add the resources to the FHIR server.

For example, as part of your study, the patient generates data at least daily the following FHIR
resources:

- MedicationAdministration
- Observation
- QuestionnaireResponse

For an example of uploading the MedicationAdministration FHIR resource to the FHIR repository, see the [FHIR POST example ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://ibm.box.com/s/31p6aj3z42ror4k8om6jeukserl52a5k){: new_window}.

For this example, keep the following information in mind that the MedicationAdministration resource includes a number of extensions to the standard MedicationAdministration resource described in the FHIR DSTU2 specification. For the sake of simplicity, only required extensions are shown. For more information about FHIR resources, see the [FHIR DSTU2 Resource list ![External link icon](../../icons/launch-glyph.svg "External link icon")](http://hl7.org/fhir/DSTU2/resourcelist.html){: new_window}.

For all FHIR examples, the `client key` and `client cert` are the key and certificate files of the client application that is signed and issued by the {{site.data.keyword.prodgroupname_short}} Certificate Authority.

## Use case 2: Adding a FHIR Bundle to the FHIR repository
{: #concept_d5b_djm_p1b__section_pfy_r34_jbb}

You can include multiple FHIR resources into a single transaction that is called a "Bundle." For example, to include the MedicicationAdministration, Observation, and QuestionnaireResponse FHIR resources as a bundle, see the [FHIR POST Bundle example ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://ibm.box.com/s/y1h3odopp6r8x3jepl5a6vl5pdhq3b6a){: new_window}.

## Use case 3: Deleting FHIR data
{: #concept_d5b_djm_p1b__d71e145}

After you add a FHIR resource to the FHIR repository, you cannot delete it. However, the DELETE API provides a "soft delete," where the DELETE API call creates a new version of the resource, along with a 'deleted' indicator.

Note: When you are deleting a file, use the <code>IBM-WH-reasonforchange</code> header to add information to the audit trail.

For more information, see the FHIR specification delete documentation at [https://www.hl7.org/fhir/DSTU2/http.html#delete ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/http.html#delete){: new_window}.

```
curl -X DELETE \
 https://watson-health.example.com/fhir-server/api/v1/Practitioner/36ee9f16-30d9-4d0b-bda6-64823405eae1 \
  --key "${HOSTNAME}.nopass.key.pem" \
  --cert "${HOSTNAME}.cert.pem" \
  -H "IBM-App-User: test_user" \
  -H "IBM-DP-correlationid: test_correlation" \
  -H "IBM-WH-reasonforchange: Deleted per patient request."
```
{: screen}

## Use case 4: Searching FHIR data
{: #concept_d5b_djm_p1b__section_g3q_xqm_z1b}

As described in the FHIR specification, you can use the FHIR GET API to find and return records based on criteria that you provide. For more information about searching, see the FHIR specification search documentation at [https://www.hl7.org/fhir/DSTU2/search.html ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/search.html){: new_window}.

For example, to find all of the questionnaire responses from a specified patient (in a specified study), enter the following curl command.

```
curl  -X GET \
  https://watson-health.example.com/fhir-server/api/v1/QuestionnaireResponse/?ptnIDstdID:exact=43541614-816a-44eb-9376-b560254af76b_study042 \
    --key "${HOSTNAME}.nopass.key.pem" \
    --cert "${HOSTNAME}.cert.pem" \
    -H "IBM-App-User: test_user" \
    -H "IBM-DP-correlationid: test_correlation"
```
{: screen}

To return detailed information about a medication, you can use the medication code, as follows:

```
curl -X GET \
https://watson-health.example.com/fhir-server/api/v1/Medication/?code=745750 \
 -H "IBM-App-User: test_user" \
 -H "IBM-DP-correlationid: test_correlation"
```
{: screen}

## Use case 5: Adding and retrieving file data to the data lake
{: #concept_d5b_djm_p1b__data_lake_usecase}

File data is any data that is not a FHIR resource, for example, HL7 data, CSV files, PDFs, or image files. You can use the {{site.data.keyword.prodname_dl}} REST APIs to upload non-FHIR data directly into the data lake.

For example, for the inhaler study, large amounts of basic information for each patient is stored in an EHR that uses HL7 format. The HL7 data is useful to track patient adherence and for other purposes.

Before you can use the {{site.data.keyword.prodname_dl}} API, you must complete a few setup steps:

1. From your {{site.data.keyword.prodgroupname_short}} Consultant, request a key and certificate for each of your client application. The key and certificate files are signed and issued by the {{site.data.keyword.prodgroupname_short}} Certificate Authority and include CN (Common Name) and OU (Organization Unit) values for your organization.
1. After you have the keys and certificates, work with your {{site.data.keyword.prodgroupname_short}} Consultant to open a ticket with IBM IT Operations (ITOps) to onboard your CN information to the data lake service. After this step is complete, you can call the {{site.data.keyword.prodname_dl}} APIs to upload and download data.

To upload data into the data lake, enter the following command. This command assumes that the client application is uploading an HL7 file called `testdata.txt`.

```
curl -X POST \
   https://watson-health.example.com/datalake/api/v1/data \
     --key "${HOSTNAME}.nopass.key.pem" \
     --cert "${HOSTNAME}.cert.pem" \
     -H "IBM-Filename: test_file" \
     -H "IBM-App-User: test_user" \
     -H "IBM-DP-correlationid: exsmple_correlationId" \
     -H "content-type: application/octet-stream"  \
     --data-binary "@testdata.txt"
```
{: screen}

When the file is uploaded, {{site.data.keyword.wh_prodname_short}} assigns a unique ID (UID), for example, `1bc5201d5c24401d8adf48a9fe7a86b6`. If you want to access or download this file, be sure to save the UID, along with the associated file name.

In addition to uploading data to the data lake, you can also download data that is already in the data lake. To download a file from the data lake, enter the following command. The download requires the UUID that was received when you uploaded the data.

```
curl -X GET \
    https://watson-health.example.com/datalake/api/v1/data/1bc5201d5c24401d8adf48a9fe7a86b6 \
     --key <client key> --cert <client cert> \
     -H "IBM-App-User: test_user" \
     -H "IBM-DP-correlationid: example_correlationId"
```
{: screen}

## Use case 6: Retrieving FHIR data from the data lake
{: #concept_d5b_djm_p1b__section_jxm_3ll_pdb}

FHIR data is added to the data lake from the FHIR repository. Use the {{site.data.keyword.wh_prodname_short}} {{site.data.keyword.prodname_dle_caps}} service to export FHIR resources for a clinical study directly from the data lake. To export data, you need to first generate an export ID and then export the data by its export ID.

The following example is an API call to generate an export request for data from an Observation FHIR resource.

```
curl -X POST \
  "https://watson-health.example.com/lake/api/v2/export?study=TestStudy1&site=TestSite1&resource=Observation&from=2018-03-11T00:01:01Z&to=2018-03-18T23:59:59Z&type=studysite" /
   --key ${HOSTNAME}.nopass.key.pem /
   --cert ${HOSTNAME}.cert.pem /
   -H "Accept: application/json" /
   -H "IBM-APP-USER: test_studyuser_email" /
   -H "IBM-APP-USER-INTID: test_studyuser" /
   -H "IBM-APP-USER-ROLE: CRO_ADMIN" /
   -H "IBM-DP-CORRELATIONID: example-correlationId"
```
{: screen}

The POST resource returns an exportId that you include as a path parameter in the GET call, as shown in the following example:

```
curl -X GET \
  "https://example.watson-health.local/lake/api/v2/export/b0585503a8bf4d0bb34b841e775580da?type=studysite&study=TestStudy1&site=TestSite1&page=1&pagesize=5&category=Exercise" /
   --key ${HOSTNAME}.nopass.key.pem /
   --cert ${HOSTNAME}.cert.pem /
   -H 'Accept: application/json' /
   -H 'IBM-APP-USER: test_studyuser_email' /
   -H 'IBM-APP-USER-INTID: test_studyuser' /
   -H 'IBM-APP-USER-ROLE: CRO_ADMIN'
   -H 'IBM-DP-CORRELATIONID: example-correlationId'
```
{: screen}

## Use case 7: Adding data to the data reservoir
{: #concept_d5b_djm_p1b__section_lqr_x11_q1b}

FHIR data is masked before it is sent to the data reservoir. However, you can also add your own data to the data reservoir. Note that it is incumbent upon the study designer to ensure that any data added to the data reservoir does not contain any PHI and is anonymous.

Before you can use the {{site.data.keyword.prodname_dr}} API, you need a user ID and password that is managed by a third-party Identity Management service. For information about getting a user ID and password, talk to your {{site.data.keyword.prodgroupname_short}} Consultant. After you obtain your user ID and password, you can use them to invoke the {{site.data.keyword.prodname_dr}} APIs.

As part of this asthma inhaler study, researchers want to include weather data from a previous study. The data is stored in CSV files and is already anonymized, so it can safely be added to the data reservoir.

To add file data to the data reservoir, enter the following command:

```
curl -X POST  \
     --data-binary "@study_1247_weather_data.csv" https://watson-health.example.com/datareservoir/api/v1/data \
      --user <user id>: \
      -H "IBM-App-User: test_user" \
      -H "IBM-DP-correlationid: example_correlationId" \
      -H "IBM-Filename: weather_data" \
      -H "content-type: application/octet-stream"
```
{: screen}

Note: If you do not include a password, the user is prompted for one when the API runs. When the file is uploaded, {{site.data.keyword.wh_prodname_short}} assigns a UUID such as <code>w1-1bc5201d5c24401d8adf48a9fe7a5498</code>. If you want to access or download this file, you need the UUID, which you can retrieve by using the metadata resource.

Note: You can use a number of methods to validate that data is uploaded correctly. One method is to use checksums, as follows:

1. Calculate a checksum before you upload the file.
1. When you upload the file, a checksum is returned in the response.
1. Compare the two checksums to validate that the data is uploaded correctly.

## Use case 8: Downloading data from the data reservoir
{: #concept_d5b_djm_p1b__section_q4d_3zv_x1b}

Data that is uploaded to the data reservoir can also be downloaded back to external files. For example, use the following command to download the CSV file that you previously uploaded from the data reservoir.

```
curl -X GET \
    https://watson-health.example.com/datareservoir/api/v1/data/w1-1bc5201d5c24401d8adf48a9fe7a5498\
      --user <user id>: \
      -H "IBM-App-User: test_user" \
      -H "IBM-DP-correlationid: example_correlationId" \
     --output c: \mydirectory\download_study_1247_data.csv
```
{: screen}

Note: Use the --output (-o) parameter to specify a download directory and file
name as part of the request.  If you do not specify a directory, then the downloaded data is saved to the directory from where the curl request was run.

## Use case 9: Retrieving files with the metadata resource
{: #concept_d5b_djm_p1b__section_lmp_mrj_rbb}

Data that is uploaded to the data reservoir can also be retrieved for a specified time period, for a specified user, or by file name.

Use the following command to return the UUID of the study_1247_weather_data.csv file that was uploaded to the data reservoir. You can then use the UUID to download the file. If you don't know when the file was uploaded, you can use a long range for fromWhen and toWhen.

```
curl -X GET \
https://watson-health.example.com/datareservoir/api/v1/metadata?fileName=study_1247_weather_data.csv?fromWhen=2010-01-01T00:00:00:Z&toWhen=2017-12-31T00:00:00Z \
    --user <user id>: \
    -H "IBM-App-User: test_user" \
    -H "IBM-DP-correlationid: example_correlationId"
```
{: screen}

Where fromWhen and toWhen parameters are the date and time are the begin and end times to retrieve records. Date must be in the following format: yyyy-MM-dd**T**hh:mm:ss**Z,**where time is in Coordinated Universal Time. fromWhen and toWhen are required.

Use the following command to return all (up to the maximum of 1000) records that were uploaded by user 12345 during November 2017:

```
curl -X GET \
  https://watson-health.example.com/datareservoir/api/v1/metadata?userId=12345?fromWhen=2017-11-01T00:00:00:Z&toWhen=2017-12-01T00:00:00Z \
    --user <user id>: \
    -H "IBM-App-User: test_user" \
    -H "IBM-DP-correlationid: example_correlationid"
```
{: screen}

Use the following command to retrieve the first 200 files in the data reservoir that were ingested between 4:00 and 4:15 AM on November 10, 2017:

```
curl -X GET \
   https://watson-health.example.com/datareservoir/api/v1/metadata?fromWhen=2017-11-10T04:00:00:Z&toWhen=2017-11-10T04:15:00Z&maxResults=200 \
    --user <user id>: \
    -H "IBM-App-User: test_user" \
    -H "IBM-DP-correlationid: example_correlationid"
```
{: screen}

Where maxResults (optional) specifies the maximum number of results, up to 1000, to return. If not specified, 1000 results are returned.
