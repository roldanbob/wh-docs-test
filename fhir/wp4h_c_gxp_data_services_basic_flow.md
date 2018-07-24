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

 # Basic Flow Use {{site.data.keyword.wh_prodname_long}} data services to upload data from sources such as hospitals, clinics, and medical devices, and make that data available for analysis and research projects. For predictive studies, data is masked and can be accessed through Spark jobs and Hive queries. In addition, for patient case management, drug trials, and other uses that require complete patient data, selected data can be sent to the {{site.data.keyword.prodname_pde_warehouse}} by using the {{site.data.keyword.prodname_pde_caps}} APIs.

## Data services detailed overview
{: #concept_lrp_gjm_p1b__section_hhj_rz5_s1b}

Data services comprise three interconnected services that, used together, manage data that is needed for scientific and medical studies. In keeping with GxP regulatory requirements, all actions within each service are audited and an audit trail is created and stored in the Audit data store.

- **FHIR service**

    Use the FHIR service to upload FHIR-formatted data into the FHIR repository. The FHIR repository consists of multiple data stores that manage your FHIR data logically. Every FHIR repository uses the following data stores:
    - Each FHIR repository includes four study data stores. Clinical studies and commercial programs each have high- and low-volume data stores.

        - High volume study data stores (HBase) contain study or program data that can change frequently (multiple times daily).
        - Low volume study data stores ( {{site.data.keyword.prodname_db2}} ) contain study and data that does not change frequently, or that can be used retrospectively. Low-volume data includes FHIR resources such as CarePlan, Contract, Device, DeviceComponent, DeviceMetric, and MedicationOrder.

    - Profile data contains the Patient FHIR resource and related information about the patient (that is, the patient's profile). To protect PHI, the Patient resource cannot be accessed or used in predictive studies. However, data from the Profile data store can be used by the {{site.data.keyword.prodname_pde_caps}} service.
    - Reference data store (Low volume) - Contains reference information for studies. Reference FHIR resources include Group, Medication, Organization, Practitioner, Questionnaire, Virtual, and Binary resources. Virtual Resources can include custom and other data that is not forwarded to the data lake.

    FHIR resources move through the {{site.data.keyword.wh_prodname_short}} pipeline in the following steps:
    - As data is written to the FHIR repository, the FHIR service calls the Consent Manager and Role and Relationship Access Manager services. These services help ensure that the patient has consented to share their information and that the person that is requesting information has the appropriate permissions. If an active consent (FHIR contract resource) record is not on file for the specified study or program, the data that is associated with the study or program is not uploaded. A 403 error (forbidden) is returned if you try to create (POST) or update (PUT) study- or program-scoped data.
    - After you upload the FHIR resources into the FHIR repository, the replicator takes the following steps:

        - Copies FHIR resources from the repository to the data lake.
        - Manages the audit trail from all data services.
        - To help ensure high availability and data resiliency, all data is replicated by the replicator.

    - From the data lake,

        - Data is sent to the {{site.data.keyword.prodname_pde_caps}} service, which provides authorized users, such as clinical investigators, with access to FHIR data. While data available to analyst users is always masked, data in the {{site.data.keyword.prodname_pde_warehouse}} can include identified patient information. Clinical researchers and care managers can use this data to design applications that can respond directly to individual patient needs such as tracking blood sugar, exercise, or asthma inhaler usage. For more information about the {{site.data.keyword.prodname_pde_caps}} service, see the [GxP {{site.data.keyword.prodname_pde_caps}} service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWH8G9qNwiyMWyltT-DL){: new_window}.
        - Authorized users can use the {{site.data.keyword.prodname_dle_caps}} service to export FHIR data from the data lake for use in clinical studies. For more information, see [{{site.data.keyword.prodname_dle_caps}} service](#concept_lrp_gjm_p1b__data_lake_export).
        - The {{site.data.keyword.prodname_de_id}} picks up the FHIR resources (except for the Patient resource) and performs a number of steps to mask and anonymize the data.

    - After masking, the {{site.data.keyword.prodname_de_id}} moves the anonymous data into the data reservoir. Data can be in the form of parquet files (the default) or JSON.
    - The masked FHIR data is sent to the data reservoir as Parquet files. From the data reservoir, analyst users can create Spark jobs or Hive queries from the FHIR data to analyze data for their studies. For more information about analyzing data from the data reservoir, see the {{site.data.keyword.wh_prodname_short}}: Analyst User Guide for the Analytics Subsystem.

- **{{site.data.keyword.prodname_dl}} service**

    The data lake contains two repositories: a file data lake for non-FHIR (file) data and a FHIR data lake for FHIR data that is sent from the FHIR repository. File data in the data lake is not used by the {{site.data.keyword.wh_prodname_short}} , and is never sent to the data reservoir. However, you can still use the {{site.data.keyword.prodname_dl}} REST API to store files in their original format and retrieve them for use with source applications.

    For file data in the data lake, the following rules apply:
    - Any data, such as CSV, binary, or HL7 data can be added to the data lake, although individual files are limited to 10 MB. If you try to upload a file larger than the maximum file size, the service, the upload is rejected with the 413 `Request Too Large` response.
    - The data lake does not validate or process file data.
    - File data in the data lake can include protected health information (PHI) such as patient names and addresses.
    - You can also use the {{site.data.keyword.prodname_dl}} REST API to retrieve file data from the data lake. To download data from the file data lake, the UUID for the file you want to download is required.

    FHIR data is added to the data lake from the FHIR repository (from the replicator). From the data lake, FHIR data is moved to the data reservoir and to the {{site.data.keyword.prodname_pde_warehouse}}.
    - To prepare studies for analysis, an analyst user needs access to data that is entirely anonymous and can be used in Spark jobs or Hive queries. To ensure that data is anonymous, FHIR data from the data lake is sent to the {{site.data.keyword.prodname_de_id}} before it is stored in the data reservoir.
    - For clinical research and care management purposes, data is moved, with PHI, to the {{site.data.keyword.prodname_pde_warehouse}}. From the {{site.data.keyword.prodname_pde_warehouse}} , the data is made available to authorized users.

    In addition, authorized clinical researchers can use the {{site.data.keyword.prodname_dle_caps}} service to package and export FHIR data directly from the data lake.

- **{{site.data.keyword.prodname_dle_caps}} service**

    The {{site.data.keyword.prodname_dle_caps}} service allows authorized users, such as clinical researchers, to export FHIR data directly from the data lake. You can use the {{site.data.keyword.prodname_dle_caps}} APIs to export data that was added to the data lake from the FHIR repository. To export data, you need to create a query that specifies a {{site.data.keyword.study}}, start and end date, and the FHIR resource type. For more information, see [{{site.data.keyword.prodname_dle_caps}} API](#concept_lrp_gjm_p1b__section_gyr_zd4_ndb). Note: To run the {{site.data.keyword.prodname_dle_caps}} APIs, you must have the CRO_ADMIN role.

- **{{site.data.keyword.prodname_dr}} service**

    Similar to the data lake, the data reservoir contains both a FHIR data reservoir and a file data reservoir. FHIR data is moved to the FHIR data reservoir from the {{site.data.keyword.prodname_de_id}}. From the FHIR data reservoir, these FHIR resources are available to user analysts from Spark jobs and through Hive queries. You can also use the {{site.data.keyword.prodname_dr}} API to upload data directly into the file data reservoir. Data that is added directly to the data reservoir is stored in the file data reservoir and is not processed in any way. If you upload data directly (rather than from the FHIR repository), note the following caveats:
    - It is incumbent on the study designer and analyst user to ensure that any data in the data reservoir is anonymized.
    - Data in the file data reservoir can be accessed by Spark for inclusion in Jupyter Notebooks. However, it is not available for Hive queries.
    - The maximum file size is 500 MB. If you try to upload a file larger than the maximum file size, the upload is rejected with the 413 (Request Too Large) response. For files that are larger than 500 MB, you can split files into smaller segments, as shown in the following example.

        1. From Linux or Cygwin, use a `split` command on a CSV file to create segments. For example, the command `split -l 50000` splits a long file into multiple files of 50000 lines each. Similar routines exist for SAS, Excel, and other data types. Note: Be sure to set the number of lines such that the size of each file is less than the stated maximum.
        1. Upload the smaller files into the file data reservoir by using the {{site.data.keyword.prodname_dr}} API.
        1. Then, from Jupyter Notebook use R (or Python) to create a single dataframe from the multiple files that were written into the data reservoir. If the files are written to a single directory (as recommended), you can create and run an R script to write the files back into a single dataframe (called `dfname`), as follows: `library(data.table) files <- list.files(path = "/etc/dump",pattern = ".csv") temp <- lapply(files, fread, sep="," dfname <- rbindlist( temp )` Note: This example shows one way to use an R script to write multiple files back into a single file or dataframe. However, many other methods in either R or Python can achieve the same result.

## Data Services REST APIs
{: #concept_lrp_gjm_p1b__section_mk5_cn1_q1b}

To upload (POST) data into the {{site.data.keyword.wh_prodname_short}} , you can use APIs to specify the data that you want to include. The APIs that you use depend on the type of data you are adding to the data lake or the data reservoir. The types of data can be broadly divided in to FHIR resources and other data (referred to here as file data). As shown in Figure 1, use the data services REST APIs for the following tasks:

- Use the FHIR API to upload FHIR resources to the FHIR repositories.
- Use the {{site.data.keyword.prodname_dl}} API to upload file data directly into the data lake.
- Use the {{site.data.keyword.prodname_dr}} API to upload file data directly into the data reservoir.

In addition, you can use these APIs to download (GET) data into external files.

- **FHIR service API**

    FHIR resources are strictly formatted and designed to be shared between healthcare entities (such as between a patient's primary care physician, cardiologist, and a hospital system). FHIR resources come from many sources, such as patient health records, medical devices (such as blood pressure monitors or scales), wearable fitness trackers, or other sources (such as weather data).

    The {{site.data.keyword.wh_prodname_short}} FHIR service API supports the following resources as described in [Table 1](#concept_lrp_gjm_p1b__table_adc_grw_qbb).

<table id="concept_lrp_gjm_p1b__table_adc_grw_qbb">
<caption>Table 1. Supported FHIR Resources.. Describes the FHIR resource types that are supported for Watson Platform for Health GxP.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e377">FHIR Resource Type</th>
<th valign="top" id="d65e380">Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e377">Binary</td>
<td valign="top" headers="d65e380">Binary information that is provided from documents such as PDFs or images.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">CarePlan</td>
<td valign="top" headers="d65e380">Patient care plan details.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Communication</td>
<td valign="top" headers="d65e380">An invitation to join a study or program as a practitioner, patient, or
administrator.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Contract</td>
<td valign="top" headers="d65e380">Patient consent information.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Device</td>
<td valign="top" headers="d65e380">A device that supports input such as a smartphone or computer, or a medical device such as an
inhaler or glucose monitor.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">DeviceComponent</td>
<td valign="top" headers="d65e380">A mobile application on a device.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">DeviceMetric</td>
<td valign="top" headers="d65e380">Quantitative or qualitative device measurements or status. User assigned.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Group</td>
<td valign="top" headers="d65e380">A Research Study group or Research Study definition.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Location</td>
<td valign="top" headers="d65e380">The residential location of a patient, guardian, or practitioner. Required to help ensure
privacy and consent rules for a country or other region.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Medication</td>
<td valign="top" headers="d65e380">Identification and definition of a medication.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">MedicationAdministration</td>
<td valign="top" headers="d65e380">Event when a patient consumes or is administered medication (such as swallowing a pill or
using an inhaler).</td>
</tr>
<tr>
<td valign="top" headers="d65e377">MedicationOrder</td>
<td valign="top" headers="d65e380">Order (prescription) for a supply of a medication and administration instructions.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">MedicationStatement</td>
<td valign="top" headers="d65e380">A record of the medication that is consumed by a patient.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Observation</td>
<td valign="top" headers="d65e380">Patient measurements and other measurements or basic information, for example:<ul id="concept_lrp_gjm_p1b__ul_uq4_mq5_xcb">
<li>Patient height and weight </li>
<li>Device observation </li>
<li>Weather information</li>
</ul>
</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Organization</td>
<td valign="top" headers="d65e380">Healthcare organization or clinical research organization, or organization site
information.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Patient</td>
<td valign="top" headers="d65e380">Patient demographics.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Person</td>
<td valign="top" headers="d65e380">Name, address, and other information about a patient, practitioner, or other user.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Practitioner</td>
<td valign="top" headers="d65e380">Researcher, medical or general wellness practitioner.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">Questionnaire</td>
<td valign="top" headers="d65e380">Structured set of questions for collecting patient information.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">QuestionnaireResponse</td>
<td valign="top" headers="d65e380">List of answers in response to a questionnaire.</td>
</tr>
<tr>
<td valign="top" headers="d65e377">RelatedPerson</td>
<td valign="top" headers="d65e380">A caregiver (guardian or third party) who is responsible for a minor or non-competent
adult.</td>
</tr>
</tbody>
</table>

A number of resource types require specific attributes or extension elements for the server to correctly process the resource. [Table 2](#concept_lrp_gjm_p1b__table_ghw_mxq_ycb) outlines the resources for which additional elements are needed. Resources that are not listed do not use elements that are outside of the base FHIR DSTU2 specification.

<table id="concept_lrp_gjm_p1b__table_ghw_mxq_ycb">
<caption>Table 2. Resources that require attributes or extension elements.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e622">ResourceType</th>
<th valign="top" id="d65e625">appName</th>
<th valign="top" id="d65e628">appVersion Number</th>
<th valign="top" id="d65e631">Composite Keys</th>
<th valign="top" id="d65e634">PatientID</th>
<th valign="top" id="d65e638">SiteID</th>
<th valign="top" id="d65e641">StudyID</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e622">Patient</td>
<td valign="top" headers="d65e625"> </td>
<td valign="top" headers="d65e628"> </td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">RelatedPerson</td>
<td valign="top" headers="d65e625"> </td>
<td valign="top" headers="d65e628"> </td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638"> </td>
<td valign="top" headers="d65e641"> </td>
</tr>
<tr>
<td valign="top" headers="d65e622">Contract</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">CarePlan</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Communication</td>
<td valign="top" headers="d65e625"> </td>
<td valign="top" headers="d65e628"> </td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634"> </td>
<td valign="top" headers="d65e638"> </td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">MedicationOrder</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">DeviceMetric</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Medication</td>
<td valign="top" headers="d65e625"> </td>
<td valign="top" headers="d65e628"> </td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634"> </td>
<td valign="top" headers="d65e638"> </td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Observation (Low-Volume)</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Observation (High Volume)</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631">X</td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Questionnaire</td>
<td valign="top" headers="d65e625"> </td>
<td valign="top" headers="d65e628"> </td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634"> </td>
<td valign="top" headers="d65e638"> </td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">ErrorLog (Virtual Resource)</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">MedicationAdministration</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631">X</td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">MedicationStatement</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631">X</td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">QuestionnaireResponse</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631">X</td>
<td valign="top" headers="d65e634">X</td>
<td valign="top" headers="d65e638">X</td>
<td valign="top" headers="d65e641">X</td>
</tr>
<tr>
<td valign="top" headers="d65e622">Location</td>
<td valign="top" headers="d65e625">X</td>
<td valign="top" headers="d65e628">X</td>
<td valign="top" headers="d65e631"> </td>
<td valign="top" headers="d65e634"> </td>
<td valign="top" headers="d65e638"> </td>
<td valign="top" headers="d65e641"> </td>
</tr>
</tbody>
</table>

Note: Calling operations for FHIR resource types that are not supported returns a 400 "Bad Request" error. The extension URIs are as follows:

    - appName - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/appName` (commercial programs only)
    - appVersionNumber - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/appVersionNumber` (commercial programs only)
    - intIdentifier - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/intIdentifier` (Contract resource only)
    - PatientID - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/patientid`
    - SiteID - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/siteID` (clinical studies only)
    - StudyID - `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/studyid`

Note:

    - appName and appVersionNumber are only required for commercial programs.
    - SiteId is only required for clinical studies.
    - The Contract resource includes multiple consent types (cloudConsent, shareDataStatement, and termsOfUse). All consent types for the Contract resource require intIdentifier. When registering a practitioner, the termsOfUse consent type does not use patientID, AppName, or AppVersionNumber.
    - For high-volume, study scoped Observation resources, the composite key is `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDrsrceNMsiteIDstdID`.
    - For MedicationAdministration, the composite key is `http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDsiteIDstdID`.

- **Methods for API calls**

    For any resource (or bundle of resources), you can call the following methods:
    - **POST [base]/fhir-server/api/v1/&lt;resource type&gt;** <dd class="dd pd">

        Upload a new resource to the FHIR repository.

    - **PUT [base]/fhir-server/api/v1/&lt;resource type&gt;/&lt;logical id&gt;** Update an existing resource.
    - **GET [base]/fhir-server/api/v1/&lt;resource type&gt;/&lt;logical id&gt;**

        Download a specific resource from the FHIR repository.

    - **GET [base]/fhir-server/api/v1/&lt;resource type&gt;/&lt;logical id&gt;/_history**

        Download the history information for a resource.

    - **GET [base]/fhir-server/api/v1/&lt;resource type&gt;/&lt;logical id&gt;/_history/&lt;version id&gt;**

        Download a specified version of a resource.

    - **DELETE [base]/fhir-server/api/v1/&lt;resource type&gt;/&lt;logical id&gt;**

        Remove a specified version of a resource.

    Where the following elements can be used within the calls:
    - `[base]` is the URL for the API call, for example: `https://demo.watson-health.local/`
    - `<resource type>` is one of the supported FHIR resources that are shown in [Table 1](#concept_lrp_gjm_p1b__table_adc_grw_qbb).
    - `<resource type properties>` are the extensions and properties that provide details about each resource type. The extensions are not explicitly called out, but, as shown in the FHIR examples, these elements are also needed for POST and PUT APIs.
    - `<logical id>` is the UUID that is uniquely identifies the resource. The logical ID is generated when the resource is POSTed to the FHIR repository.
    - `<version id>` is a UUID that uniquely identifies a specific version of a resource.

- **FHIR API response codes**

    The FHIR server returns the following HTTP response codes for HTTP requests that succeed or fail with specific error conditions.

<table id="concept_lrp_gjm_p1b__table_j2v_1cn_z1b">
<caption>Table 3. FHIR API response codes.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1223">Response code</th>
<th valign="top" id="d65e1226">Description</th>
<th valign="top" id="d65e1229">Examples</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1223">200/OK</td>
<td valign="top" headers="d65e1226">The request is successful.</td>
<td valign="top" headers="d65e1229">GET (read, vread, search)<br />PUT request (update)</td>
</tr>
<tr>
<td valign="top" headers="d65e1223">201/Created</td>
<td valign="top" headers="d65e1226">The request successfully created a new resource. </td>
<td valign="top" headers="d65e1229">POST<br />PUT request (create new resource)</td>
</tr>
<tr>
<td valign="top" headers="d65e1223">204/No content. </td>
<td valign="top" headers="d65e1226">The operation completed successfully but there is no data to return.</td>
<td valign="top" headers="d65e1229">GET (deleted resource). After a resource is deleted, the resource cannot be found. </td>
</tr>
<tr>
<td valign="top" headers="d65e1223">400/Bad Request</td>
<td valign="top" headers="d65e1226">The request failed due to malformed syntax. Correct the syntax before you rerun the request. </td>
<td valign="top" headers="d65e1229">Invalid request (for example, FHIR resources malformed), unsupported FHIR resource type, or
invalid API parameters. </td>
</tr>
<tr>
<td valign="top" headers="d65e1223">401/Unauthorized</td>
<td valign="top" headers="d65e1226">The request requires user authentication. Either user credentials are required or the
provided credentials are invalid. </td>
<td valign="top" headers="d65e1229">User is not authenticated or not authorized to access the requested resource or API.</td>
</tr>
<tr>
<td valign="top" headers="d65e1223">403/Forbidden</td>
<td valign="top" headers="d65e1226">The server understood the request but cannot fulfill it. </td>
<td valign="top" headers="d65e1229">Active consent is not found for the patient in the context of a POST or PUT request for data
for a particular study. </td>
</tr>
<tr>
<td valign="top" headers="d65e1223">404/Object not found. </td>
<td valign="top" headers="d65e1226">The requested object does not exist or cannot be found.</td>
<td valign="top" headers="d65e1229">Invalid request (for example, where the resource ID does not exist). </td>
</tr>
</tbody>
</table>

## Additional FHIR details

- **FHIR API headers**

    HTTP headers are required for the FDA CFR Part 11 Audit trail. All create, update, and delete operations must contain the following information:

<table id="concept_lrp_gjm_p1b__table_wp2_x51_tcb">
<caption>Table 4. FHIR API request headers</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1360">FHIR API header</th>
<th valign="top" id="d65e1363">Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1360">IBM-App-User</td>
<td valign="top" headers="d65e1363">The external ID (the user name) of the authenticated user who is making the API request.
</td>
</tr>
<tr>
<td valign="top" headers="d65e1360">IBM-App-User-IntId</td>
<td valign="top" headers="d65e1363">The internal ID (the user ID) associated with the IBM-App-User value.</td>
</tr>

<tr>
<td valign="top" headers="d65e1360">IBM-DP-correlationid</td>
<td valign="top" headers="d65e1363">A unique ID that correlates requests across the client application and the service.</td>
</tr>
<tr>
<td valign="top" headers="d65e1360">IBM-WH-reasonforchange</td>
<td valign="top" headers="d65e1363">An optional "reason for change" text field to store in the audit trail reason field. The
accepted value is a string of up to 255 printable US-ASCII characters.</td>
</tr>
<tr>
<td valign="top" headers="d65e1360">X-WHC-LSF-resourcename</td>
<td valign="top" headers="d65e1363">A valid resourceName (within a resourceType). Required for "read", "vread", "history" and
"delete" operations. The resourceName is used to route the request to the correct data
store.</td>
</tr>
<tr>
<td valign="top" headers="d65e1360">X-WHC-LSF-rolename </td>
<td valign="top" headers="d65e1363">The role that is assigned to the IBM-App-User. If not specified, the default role is
Patient.</td>
</tr>
<tr>
<td valign="top" headers="d65e1360">X-WHC-LSF-studyid</td>
<td valign="top" headers="d65e1363">The ID of this study (for study-scoped resources). </td>
</tr>
</tbody>
</table>

- **Using Bundles**

    Resources can be bundled together (in a single POST request) which allows for efficiencies in terms of batch processing and ingestion of resources. Bundles can optionally be persisted as a single atomic unit of work by using transaction bundles (that is, where Bundle.type = transaction). See [Use case 2: Adding a FHIR Bundle to the FHIR repository](#d71e139) for an example that uses a bundle.

    For more information about bundles, see [https://www.hl7.org/fhir/DSTU2/bundle.html ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/bundle.html){: new_window}. Note: The Patient, Contract, and Basic resources are not forwarded to the data lake. For the {{site.data.keyword.prodname_pde_caps}} ({{site.data.keyword.prodname_adherence}}) service, a custom resource that contains study data is forwarded to the {{site.data.keyword.prodname_adherence}} consumer.

- **Removing data from the FHIR repository**

    The DELETE API call performs a soft delete. That is, when a resource is deleted, the FHIR service creates a new version of the resource, along with a 'deleted' indicator. This indicator does not exist within the resource JSON itself, but is persisted along with the resource to indicate to the persistence layers that the resource is no longer available. For an example of deleting a FHIR resource, see [Use case 3: Deleting FHIR data](#d71e148). For more information about deletion with FHIR resources, see the FHIR specification delete documentation at [https://www.hl7.org/fhir/DSTU2/http.html#delete ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/http.html#delete){: new_window}.

    For more information about the FHIR API, see the [GxP FHIR API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/gxp_fhirserver_swagger.json){: new_window} documentation.

## Using the FHIR Registration API (custom operations)
{: #concept_lrp_gjm_p1b__section_bkr_dg2_5db}

The FHIR Registration API provides a set of methods to manage the registration for patients,
practitioners, and other people who are involved in a patient's care for clinical studies and
commercial programs.

- For clinical studies, patients and practitioners register directly with the study through the `registerPatient` or `registerPractioner` methods.
- After a patient or practitioner registers for a commercial program, they generally receive an email invitation. To join the commercial program, they need to take the additional step of accepting the invitation, which is managed through the `acceptPatientInvitation` or `acceptCareManagerInvitation` methods.
- For patients who are minors or non-competent adults, the `registerThirdParty` method allows a guardian (for minors) or a third party (for non-competent adults) to establish a relationship with a patient. The guardian or third party has access to a patient's healthcare data and history.
- The remaining methods allow administrators to manage patient and practitioners and their relationships:

    - The `removeRelationship` method removes a third-party or guardianship relationship.
    - The `removePatient` and `removePractitioner` methods remove a patient or practitioner from a clinical study or commercial program.

Details for the Registration API are available in the [GxP FHIR API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/gxp_fhirserver_swagger.json){: new_window}.

## Searching FHIR resources

You can use the GET API resource to search for and return FHIR resources from the FHIR repository. For more information about the supported search features, see [https://www.hl7.org/fhir/DSTU2/search.html ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU/search.html){: new_window}. For an example, see [Use case 4: Searching FHIR data](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html#d71e164).

- **Supported search parameters**

    For the FHIR resources supported by {{site.data.keyword.wh_prodname_short}} , the FHIR REST API supports the following search parameters:
    - _id
    - _lastUpdated
    - _profile
    - _tag
    - _security

    In addition, you can use the following search result parameters:
    - _count
    - _page

    Note:

     The _count search result parameter returns up to 1000 records. If a _count of greater than

    1000 is requested, only the first 1000 records are returned.

    The following search parameters are supported for all study-scoped resources:
    - patientid - Optional.
    - siteid - Optional.
    - studyid – Required for searches on study-scoped resources. For reads and vreads, X-WHC-LSF-studyid is a required HTTP header. Note: When searching on the Medication resource, you can specify patientid and studyid, but not siteid.

- **Supported search prefixes**

    You can use the following prefixes with number, date, and quantity search parameter types:
    - eq – Equal to (default). When no prefix is specified, the default is an exact match (equal to) search.
    - ne – Not equal to
    - gt – Greater than
    - lt- Less than
    - ge – Greater than or equal to
    - le – Less than or equal to

- **Search modifiers**

    {{site.data.keyword.wh_prodname_short}} supports the following search modifiers.

<table id="concept_lrp_gjm_p1b__table_i33_yym_z1b">
<caption>Table 5. Supported FHIR search modifiers</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1695">FHIR search parameter</th>
<th valign="top" id="d65e1698">Supported modifiers</th>
<th valign="top" id="d65e1701">Search behavior (without modifier)</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1695">String</td>
<td valign="top" headers="d65e1698">:exact, :contains</td>
<td valign="top" headers="d65e1701">A "starts with" search. Case- and accent-insensitive.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">Reference</td>
<td valign="top" headers="d65e1698">:[type]</td>
<td valign="top" headers="d65e1701">An exact match search.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">URI</td>
<td valign="top" headers="d65e1698">:below</td>
<td valign="top" headers="d65e1701">A "starts with" search.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">Token</td>
<td valign="top" headers="d65e1698">:below, :not</td>
<td valign="top" headers="d65e1701">An exact match search.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">Number</td>
<td valign="top" headers="d65e1698">None</td>
<td valign="top" headers="d65e1701">If present, uses prefix, else an exact match search.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">Date</td>
<td valign="top" headers="d65e1698">None</td>
<td valign="top" headers="d65e1701">If present, uses prefix, else an exact match search.</td>
</tr>
<tr>
<td valign="top" headers="d65e1695">Quantity</td>
<td valign="top" headers="d65e1698">None </td>
<td valign="top" headers="d65e1701">If present, uses prefix, else an exact match search.</td>
</tr>
</tbody>
</table>

Note: Modifiers that are not supported return a 400 `Bad Request` error.

- **Searching on data/time values**

    When time is provided, the service requires values for hour, minute, second, and time zone as part of the entire date/time value.

- **Working with study-scoped FHIR resources**

    Study-scoped data is information for which patient consent is required (and available). Information such as patient name and address, medical history, and device ID is protected and can be accessed only by authorized users. A platform-specific HTTP header is required to retrieve or update study-scoped data when you use the API patterns that are shown in [Table 6](#concept_lrp_gjm_p1b__table_xjs_wrn_z1b).

<table id="concept_lrp_gjm_p1b__table_xjs_wrn_z1b">
<caption>Table 6. API patterns for study scoped resources.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1834">Action </th>
<th valign="top" id="d65e1837">API patterns</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1834">create</td>
<td valign="top" headers="d65e1837">POST [base]/[resource type] </td>
</tr>
<tr>
<td valign="top" headers="d65e1834">read</td>
<td valign="top" headers="d65e1837">GET [base]/[resource type]/[logical id] </td>
</tr>
<tr>
<td valign="top" headers="d65e1834">update</td>
<td valign="top" headers="d65e1837">PUT [base]/[resource type]/[logical id] </td>
</tr>
<tr>
<td valign="top" headers="d65e1834">delete</td>
<td valign="top" headers="d65e1837">DELETE [base]/[resource type]/[logical id] </td>
</tr>
<tr>
<td valign="top" headers="d65e1834">vread</td>
<td valign="top" headers="d65e1837">GET [base]/[resource type]/[logical id]/_history/[version id] </td>
</tr>
<tr>
<td valign="top" headers="d65e1834">history</td>
<td valign="top" headers="d65e1837">GET [base]/[resource type]/[logical id]/_history{?[parameters]}</td>
</tr>
<tr>
<td valign="top" headers="d65e1834">search</td>
<td valign="top" headers="d65e1837">GET [base]/[resource type] {?[parameters]}</td>
</tr>
</tbody>
</table>

When you retrieve or update study-scoped data with the patterns that are described in [Table 6](#concept_lrp_gjm_p1b__table_xjs_wrn_z1b) a study ID must be present. The valid locations are as shown in [Table 7](#concept_lrp_gjm_p1b__table_jdt_3t5_z1b).

<table id="concept_lrp_gjm_p1b__table_jdt_3t5_z1b">
<caption>Table 7. Study ID locations for study-scoped resources.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1932">FHIR Operation</th>
<th valign="top" id="d65e1935">Study ID Location</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1932">create, update</td>
<td valign="top" headers="d65e1935">studyid extension element in resource body</td>
</tr>
<tr>
<td valign="top" headers="d65e1932">search (low volume resources)</td>
<td valign="top" headers="d65e1935">studyid query string parameter (e.g. Device?studyid=STUDY01)</td>
</tr>
<tr>
<td valign="top" headers="d65e1932">read, vread (version read), history, search (high-volume resources), delete</td>
<td valign="top" headers="d65e1935">X-WHC-LSF-studyid header</td>
</tr>
</tbody>
</table>

    For search requests that conform to the supported API patterns, you must use the studyid URL parameter.

    ```
    GET [base]/[type]{?[parameters]}
    POST [base]/[type]/_search{?[parameters]}
    ```
    {: screen}

    In the following scenarios, include the query string parameters that are shown in [Table 8](#concept_lrp_gjm_p1b__table_oww_p55_z1b) in FHIR search requests (query string) for study-scoped resources:

<table id="concept_lrp_gjm_p1b__table_oww_p55_z1b">
<caption>Table 8. Request parameters for query strings.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e1994">Request parameter</th>
<th valign="top" id="d65e1997">Scenario</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e1994">studyid</td>
<td valign="top" headers="d65e1997">Required for low-volume, study-scoped resources stored in DB2.</td>
</tr>
<tr>
<td valign="top" headers="d65e1994">A single "compound key" query string parameter whose value is a concatenation of other fields
(such as. patientIdSiteIdStudyId)</td>
<td valign="top" headers="d65e1997">Required for high-volume study-scoped resources. For high-volume resources, only one compound
key search parameter can be used per search request. The FHIR server returns a 400 <code>Bad
Request</code> if you specify more than one compound key in the search request.</td>
</tr>
</tbody>
</table>

- **Searching with high-volume, study-scoped resources**

    High volume resources, which are stored in HBase, have certain limitations when it comes to the FHIR search operation. Specifically, searches can include only a single compound key search parameter and an optional date/time (`timestamp`) parameter, as shown in the following example:`GET [base]/fhir-server/api/v1/Observation?compoundKey:exact=abc123xyz&date=2017-07-18T00:00:00.000Z`

    Because compound keys require an exact match, the `exact` modifier is the only legal modifier. Search requests that do not include the exact modifier are rejected with a 400 `Bad Request` error. The high-volume data store has a single notion of time stamp for each cell. Therefore, only one time stamp (date/time) can be indexed for each resource type. The time stamp fields that are chosen for high-volume resources are identified in [Table 9](#concept_lrp_gjm_p1b__table_f3h_gv5_z1b). Although time stamp (data/time) search parameters are optional, the values for these fields are required for proper indexing and are validated during ingestion on FHIR create (POST) and FHIR update (PUT) operations.

<table id="concept_lrp_gjm_p1b__table_f3h_gv5_z1b">
<caption>Table 9. Time stamps for study-scoped resources.</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e2069">Resource Type Name</th>
<th valign="top" id="d65e2072">Timestamp (date/time) field</th>
<th valign="top" id="d65e2075">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e2069">MedicationAdministration</td>
<td valign="top" headers="d65e2072">MedicationAdministration.effectiveTimeDateTime</td>
<td valign="top" headers="d65e2075">Time of administration.</td>
</tr>
<tr>
<td valign="top" headers="d65e2069">Observation</td>
<td valign="top" headers="d65e2072">Observation.effectiveDateTime</td>
<td valign="top" headers="d65e2075">Clinically relevant time for observation.</td>
</tr>
<tr>
<td valign="top" headers="d65e2069">QuestionnaireResponse</td>
<td valign="top" headers="d65e2072">QuestionnaireResponse.authored</td>
<td valign="top" headers="d65e2075">Authoring date for this version.</td>
</tr>
</tbody>
</table>

Note: Both MedicationAdministration and Observation have fields (MedicationAdminstration.effectiveTimePeriod and Observation.efffectivePeriod) that use the FHIR Period data type. Because the high volume data store has only a single notion of time, you cannot use these fields for searches. Only the fields in [Table 9](#concept_lrp_gjm_p1b__table_f3h_gv5_z1b) can be indexed for search. For more information, see the [GxP FHIR Service API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/gxp_fhirserver_swagger.json){: new_window} documentation. Note: Due to the size of the FHIR Service API, the Swagger file may take a few minutes to open.

## Data Lake API
{: #concept_lrp_gjm_p1b__section_xqg_4k5_bdb}

Use the {{site.data.keyword.prodname_dl}} REST API to perform the following tasks:

- Upload file data directly into the data lake. Data in the file data lake is never moved to the data reservoir and is not processed or validated.
- Download non-FHIR data from the data lake to external files. To download file data, the client application needs to know the UUID of the file to download.

{{site.data.keyword.prodname_dl}} APIs use certificate-based authentication and white-listing. API requests come from a client application (rather than a person, as with the {{site.data.keyword.prodname_dr}} API). The {{site.data.keyword.prodname_dl}} API takes the following steps:

1. The client application sends a request to the {{site.data.keyword.prodname_dl}} API that uses its client certificate. The client certificate must be signed by the {{site.data.keyword.prodgroupname_short}} Certificate Authority or the request is rejected.
1. The {{site.data.keyword.prodname_dl}} Server uses an internal white-list to verify the CN (Common Name) and OU (Organization Unit) values of the client certificate. The verification helps ensure that a client with the specified CN and OU is authorized to make the request.
1. Additionally, calls to the {{site.data.keyword.prodname_dl}} API require that the client application provide IBM-DP-correlationId and IBM-App-User headers, which are used as elements in audit records. The IBM-DP-correlationId specifies a unique ID to link audit records across the client application and the {{site.data.keyword.prodname_dl}} service. The IBM-App-User specifies the user information to be captured in the audit records.

Note: FHIR data is moved to the data lake from the FHIR repository by the replicator, and copied from the data lake by the {{site.data.keyword.prodname_de_id}}. For more information, see the [Data De-identification service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWIfUP6BwiyMWyltT-F_){: new_window}.

For examples, see [Use case 5: Adding and retrieving file data to the data lake](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html#d71e190).

For more information, see the [GxP {{site.data.keyword.prodname_dl}} Service API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/gxp_datalake_ext_swagger.yaml){: new_window} documentation.

## Data Lake Export API
{: #concept_lrp_gjm_p1b__section_gyr_zd4_ndb}

Use the {{site.data.keyword.prodname_dle_caps}} API to export FHIR resources directly from the data lake. Use the <code>POST /v2/export/</code> resource to query the data base to find matching FHIR resources. The {{site.data.keyword.prodname_dle_caps}} API requires the following header information:

<table id="concept_lrp_gjm_p1b__table_ljx_d32_pdb">
<caption>Table 10. Data Lake Export API request headers</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e2241">
Data Lake Export
 API header</th>
<th valign="top" id="d65e2246">Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e2241">IBM-App-User</td>
<td valign="top" headers="d65e2246">The external ID (the user name) of the authenticated user who is making the API request.
</td>
</tr>
<tr>
<td valign="top" headers="d65e2241">IBM-App-User-IntId</td>
<td valign="top" headers="d65e2246">The internal ID (the user ID) associated with the IBM-App-User value.</td>
</tr>
<tr>
<td valign="top" headers="d65e2241">IBM-App-User-Role</td>
<td valign="top" headers="d65e2246">The role assigned to the IBM-App-User. Must be CRO_ADMIN.</td>
</tr>
<tr>
<td valign="top" headers="d65e2241">IBM-DP-correlationid</td>
<td valign="top" headers="d65e2246">A unique ID that correlates requests across the client application and the service.</td>
</tr>
</tbody>
</table>

For <code>POST /v2/export/</code> create a query that specifies the following criteria:

- `resource` - The FHIR resource to return. The following resources are available for export:

<table id="concept_lrp_gjm_p1b__dle_resources">
<caption>Table 11. FHIR resources available for Data Lake Export</caption><colgroup></colgroup>
<thead style="text-align:left;">
<tr>
<th valign="top" id="d65e2313">FHIR resource</th>
<th valign="top" id="d65e2316">FHIR resource</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top" headers="d65e2313">Contract</td>
<td valign="top" headers="d65e2316">MedicationStatement</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">Communication</td>
<td valign="top" headers="d65e2316">Observation</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">Device</td>
<td valign="top" headers="d65e2316">Organization</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">DeviceComponent</td>
<td valign="top" headers="d65e2316">Patient</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">DeviceMetric</td>
<td valign="top" headers="d65e2316">Person</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">Group</td>
<td valign="top" headers="d65e2316">Practitioner</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">Medication</td>
<td valign="top" headers="d65e2316">Questionnaire</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">MedicationAdministration</td>
<td valign="top" headers="d65e2316">QuestionnaireResponse</td>
</tr>
<tr>
<td valign="top" headers="d65e2313">MedicationOrder</td>
<td valign="top" headers="d65e2316">RelatedPerson</td>
</tr>
</tbody>
</table>

- `from`/`to` - The time period for which to search for records. `from` and `to` specify a date and time to begin and end searching for records. The dates must be in the following format: yyyy-MM-dd**T**hh:mm:ss**Z,**where time is in Coordinated Universal Time.
- `type` - The type of export, which must be `studysite`.
- `site` and `study` - The site ID and study ID for which you want to return FHIR resource data.

The POST /v2/export/ resource returns an export ID that is generated by the server. To export the JSON response,specify the following criteria:

- `exportId` - The ID generated by the server for a set of FHIR resources to return.
- `pagesize` - The number of entries to return per page. The default (and maximum) is 100.
- `page` - The page to be returned. The default (and minimum) is 1.
- `resource` - The FHIR resource to return, as shown in [Table 11](#concept_lrp_gjm_p1b__dle_resources).
- `category` - For Observation resources, you must specify the observation category, which can be:

    - biometrics
    - bloodglucoseReading
    - cgm_readings (continuous glucose monitoring)
    - Exercise
    - Food
    - Height
    - Weight

- `type` - The type of export, which must be `studysite`.
- `site` and `study` - The site ID and study ID for which you want to return FHIR resource data.

For examples, see [Use Case 6: Retrieving FHIR data from the data lake](/docs/services/DATA-SERVICES-CATALOG/wp4h_c_gxp_data_services_use_cases.html#concept_d5b_djm_p1b__section_jxm_3ll_pdb).

For more information, see the [GxP V2.1.1 {{site.data.keyword.prodname_dle_caps}} Service API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](http://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/lake-export-r211.yaml){: new_window}

## Data Reservoir API
{: #concept_lrp_gjm_p1b__section_yqg_4k5_bdb}

The data reservoir contains fully de-identified data that is ready to be used for analysis. Use the {{site.data.keyword.prodname_dr}} API for the following tasks:

- Upload file data into the data reservoir.
- Download file data from the data reservoir into external files.
- Search for and retrieve files by any of the following criteria:

    - `fromWhen`/`toWhen` - Specify a time period for which to search for records. `fromWhen` and `toWhen` specify a date and time to begin and end searching for records. Date must be in the following format: yyyy-MM-dd**T**hh:mm:ss**Z,**where time is in Coordinated Universal Time.
    - `userId` - Optional. Retrieves records that were uploaded by the specified data reservoir user.
    - `filename` - Optional. Retrieve a file by its name. Be sure to specify the name of the file that is stored in the data reservoir.
    - `maxRecords` - Optional. Specify the number of records to retrieve (up to 1000). If not specified, the API returns the first (that is, oldest) 1000 records in the specified time period. Note: An authorized user can download any file that has been uploaded into the data reservoir via the {{site.data.keyword.prodname_dr}} API.

Note: If you upload data directly into the data reservoir, make sure that the data does not contain any PHI, PII, or SPI. It is the study designer's role to ensure that all data that is uploaded directly by using the {{site.data.keyword.prodname_dr}} APIs is properly de-identified and anonymous.If the file data is in a format that can be read by a Spark job, user analysts can use Spark to add the data to Jupyter Notebook. From the file data reservoir, Spark has been tested with data in CSV, JSON, Parquet, and TXT formats. Other formats may work, but have not been tested.

Note: To use data from the data reservoir in a Spark job, you need the file name of the file in the data reservoir. Be sure to note the file name when you POST the file to the data reservoir.

For more information about using Spark jobs with data in the file data reservoir, see the {{site.data.keyword.wh_prodname_short}}: Analyst User Guide for the Analytics Subsystem.

For examples, see [Use case 7 Adding data to the data reservoir](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html#d71e250), and [Use case 8: Downloading data from the data reservoir](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html#d71e270), and [Use case 9: Retrieving files using metadata](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html#d71e285).

For more information, see the [GxP {{site.data.keyword.prodname_dr}} Service API Swagger ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://wp4h-gxp.w3ibm.mybluemix.net/index.html?url=/gxp_datareservoir_ext_swagger.yaml){: new_window} documentation.
