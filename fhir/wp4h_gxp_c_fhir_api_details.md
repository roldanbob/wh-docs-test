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

---

 # Getting started with the FHIR REST API

Fast Healthcare Interoperability Resources (FHIR) is a standard for exchanging electronic information between healthcare entities (such as between a patient's primary care physician, cardiologist, and a hospital system). FHIR resources come from many sources, including patient health records, medical devices (such as blood pressure monitors or scales), wearable fitness trackers, or other sources (such as weather data). FHIR resources are strictly formatted and can be accessed via the FHIR service REST API. The {{site.data.keyword.wh_prodname_short}} FHIR service API supports the resources that are described in Table 1 of *Using the FHIR Service*. <!-- [Table 1](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_service.html#wp4h_gxp_c_fhir_service__table_ckk_scb_hbb).-->

{{site.data.keyword.wh_prodname_long}} also supports a subset of the FHIR search parameters that are described in the FHIR DSTU2 specification. In addition, to use and understand the FHIR REST API, other information about the {{site.data.keyword.wh_prodname_short}} implementation is required.

## Using the FHIR REST API
The FHIR REST API is based on the FHIR DSTU2 specification. For detailed information about this version of FHIR and the FHIR specification, see [Welcome to FHIR](http://hl7.org/fhir/DSTU2).

### FHIR API methods
{: #concept_isb_c13_1cb__section_dz1_tt1_tcb}

For any resource (or bundle of resources), you can call the following methods:

- Upload a new resource to the FHIR repository:
   `POST [base]/fhir-server/api/v1/{resource type}`
- Update an existing resource: 
   `PUT [base]/fhir-server/api/v1/{resource type}/{logical id}`
- Search for information about a specific resource from the FHIR repository:
   `GET [base]/fhir-server/api/v1/{resource type}/{search parameters}` 
   
  **Note:** {{site.data.keyword.wh_prodname_short}} supports a subset of the FHIR DSTU 2 search parameters. For more information about search parameters, see the [FHIR DSTU2 specification, Search section](http://hl7.org/fhir/DSTU2/search.html).
- Download a specific resource: 
   `GET [base]/fhir-server/api/v1/{resource type}/{logical id}`
- Download the history information for a resource: 
   `GET [base]/fhir-server/api/v1/{resource type}/{logical id}/_history`
- Download a specified version of a resource: 
   `GET [base]/fhir-server/api/v1/{resource type}/{logical id}/_history/{version id}`
- Remove a specified version of a resource: 
   `DELETE [base]/fhir-server/api/v1/{resource type}/{logical id}` 
   
   **Note:** The DELETE API performs a soft delete, that is, data is marked as deleted but not actually removed from the data store.

### FHIR API URL elements
{: #concept_isb_c13_1cb__section_n54_4xd_ycb}
The following URL elements can be used within the calls:

- `[base]` is the URI for the method, for example: `https://demo.watson-health.local/`
- `{resource type}` is one of the supported FHIR resource types or resource names that are shown in Table 1 of *Using the FHIR Service*. <!-- [Table 1](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_service.html#wp4h_gxp_c_fhir_service__table_ckk_scb_hbb). -->

  **Note:** In certain circumstances, you can specify a resourceName instead of a resourceType. For example, the Observation resourceType is associated with a number of different resourceNames, such as Observation (Device)::MedicalDeviceObservation and Observation (Patient):: ParticipantObservation (Height) . These resources are routed to different data stores; in this case, you can specify the MedicalDeviceObservation and ParticipantObservation resourceNames. The FHIR API associates the resourceName with the correct resourceType and routes the resources correctly.
- `{resource type properties}` are the extensions and properties that provide details about each resource type. The extensions are not explicitly called out, but these elements are also needed for POST and PUT methods.
- `{logical id}` is the UUID that is uniquely identifies the resource. The logical ID is generated when the resource is POSTed to the FHIR repository.
- `{version id}` is a UUID that uniquely identifies a specific version of a resource.

### FHIR API request headers

For FHIR API methods, the following request headers are available:

<table id="wp4h_gxp_c_fhir_details__table_wp2_x51_tcb">
<caption>Table 1. FHIR API request headers</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e43">FHIR API header</th>
<th id="d1817e46">Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e43">IBM-App-User</td>
<td headers="d1817e46">The external ID (the user name) of the authenticated user who is making the API request.
</td>
</tr>
<tr>
<td headers="d1817e43">IBM-App-User-IntId</td>
<td headers="d1817e46">The internal ID (the user ID) associated with the IBM-App-User value.</td>
</tr>

<tr>
<td headers="d1817e43">IBM-DP-correlationid</td>
<td headers="d1817e46">A unique ID that correlates requests across the client application and the service.</td>
</tr>
<tr>
<td headers="d1817e43">IBM-WH-reasonforchange</td>
<td headers="d1817e46">An optional "reason for change" text field to store in the audit trail
<code>reason</code> field. The accepted value is a string of up to 255 printable US-ASCII
characters.</td>
</tr>
<tr>
<td headers="d1817e43">X-WHC-LSF-resourcename</td>
<td headers="d1817e46">A valid resourceName (within a resourceType). Required for "read", "vread (version read)",
"history" and "delete" operations. The resourceName is used to route the request to the correct data
store.</td>
</tr>
<tr>
<td headers="d1817e43">X-WHC-LSF-rolename </td>
<td headers="d1817e46">The role that is assigned to the IBM-App-User. If not specified, the default role is
Patient.</td>
</tr>
<tr>
<td headers="d1817e43">X-WHC-LSF-studyid</td>
<td headers="d1817e46">The ID of this study (for study-scoped resources). </td>
</tr>
</tbody>
</table>

## Searching with FHIR

You can use the GET API to search for and return FHIR resources from the FHIR repository. {{site.data.keyword.wh_prodname_short}} supports a subset of the FHIR search features. Differences between the FHIR specification and {{site.data.keyword.wh_prodname_short}} are described here. For more information about search features, see [https://www.hl7.org/fhir/DSTU2/search.html](https://www.hl7.org/fhir/DSTU2/search.html).

### Supported search parameters

For the FHIR resources supported by {{site.data.keyword.wh_prodname_short}} , the FHIR REST API supports the following search parameters:
 * _id
 * _lastUpdated
 * _profile
 * _tag
 * _security

 In addition, you can use the following search result parameters:
 * _count
 * _page

**Note:** The _count search result parameter returns up to 1000 records. If a _count of greater than 1000 is requested, only the first 1000 records are returned.

The following search parameters are supported for all study-scoped resources:
 * patientid - Optional.
* siteid - Optional.
* studyid – Required for searches on study-scoped resources. For reads and vreads, X-WHC-LSF-studyid is a required HTTP header.
   
 **Note:** When searching on the Medication resource, you can specify patientid and studyid, but not siteid.

### Supported search prefixes

 You can use the following prefixes with number, date, and quantity search parameter types:
 * eq – Equal to (default). When no prefix is specified, the default is an exact match (equal to) search.
 * ne – Not equal to
 * gt – Greater than
 * lt - Less than
 * ge – Greater than or equal to
 * le – Less than or equal to

### Search modifiers

{{site.data.keyword.wh_prodname_short}} supports the search modifiers described in [Table 2](wp4h_gxp_c_fhir_details__table_i33_yym_z1b).

<table id="wp4h_gxp_c_fhir_details__table_i33_yym_z1b">
<caption>Table 2. Supported FHIR search modifiers</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e266">FHIR search parameter</th>
<th id="d1817e269">Supported modifiers</th>
<th id="d1817e272">Search behavior (without modifier)</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e266">String</td>
<td headers="d1817e269">:exact, :contains</td>
<td headers="d1817e272">A "starts with" search. Case- and accent-insensitive.</td>
</tr>
<tr>
<td headers="d1817e266">Reference</td>
<td headers="d1817e269">:[type]</td>
<td headers="d1817e272">An exact match search.</td>
</tr>
<tr>
<td headers="d1817e266">URI</td>
<td headers="d1817e269">:below</td>
<td headers="d1817e272">A "starts with" search.</td>
</tr>
<tr>
<td headers="d1817e266">Token</td>
<td headers="d1817e269">:below, :not</td>
<td headers="d1817e272">An exact match search.</td>
</tr>
<tr>
<td headers="d1817e266">Number</td>
<td headers="d1817e269">None</td>
<td headers="d1817e272">If present, uses prefix, else an exact match search.</td>
</tr>
<tr>
<td headers="d1817e266">Date</td>
<td headers="d1817e269">None</td>
<td headers="d1817e272">If present, uses prefix, else an exact match search.</td>
</tr>
<tr>
<td headers="d1817e266">Quantity</td>
<td headers="d1817e269">None </td>
<td headers="d1817e272">If present, uses prefix, else an exact match search.</td>
</tr>
</tbody>
</table>

**Note:** Modifiers that are not supported return a 400 `Bad Request` error.

### Searching on data/time values

When time is provided, the service requires values for hour, minute, second, and time zone as part of the entire date/time value.

### Working with study-scoped FHIR resources

Study-scoped data is information for which patient consent is required (and available). Information such as patient name and address, medical history, and device ID is protected and can be accessed only by authorized users. A platform-specific HTTP header is required to retrieve or update study-scoped data when you use the API patterns that are shown in [Table 3](#wp4h_gxp_c_fhir_details__table_xjs_wrn_z1b).

<table id="wp4h_gxp_c_fhir_details__table_xjs_wrn_z1b">
<caption>Table 3. API patterns for study scoped resources.</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e403" style="width:20%">Action </th>
<th id="d1817e406">API patterns</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e403">create</td>
<td headers="d1817e406">POST [base]/[resource type] </td>
</tr>
<tr>
<td headers="d1817e403">read</td>
<td headers="d1817e406">GET [base]/[resource type]/[logicalId] </td>
</tr>
<tr>
<td headers="d1817e403">update</td>
<td headers="d1817e406">PUT [base]/[resource type]/[logicalId] </td>
</tr>
<tr>
<td headers="d1817e403">delete</td>
<td headers="d1817e406">DELETE [base]/[resource type]/[logicalId] </td>
</tr>
<tr>
<td headers="d1817e403">vread</td>
<td headers="d1817e406">GET [base]/[resource type]/[logicalId]/_history/[versionId] </td>
</tr>
<tr>
<td headers="d1817e403">history</td>
<td headers="d1817e406">GET [base]/[resource type]/[logicalId]/_history{?[parameters]}</td>
</tr>
<tr>
<td headers="d1817e403">search</td>
<td headers="d1817e406">GET [base]/[resource type] {?[parameters]}</td>
</tr>
</tbody>
</table>

When you retrieve or update study-scoped data with the patterns that are described in [Table 3](#wp4h_gxp_c_fhir_details__table_xjs_wrn_z1b), a study ID must be present. The valid locations are as shown in [Table 4](#wp4h_gxp_c_fhir_details__table_jdt_3t5_z1b).

<table id="wp4h_gxp_c_fhir_details__table_jdt_3t5_z1b">
<caption>Table 4. Study ID locations for study-scoped resources.</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e501">FHIR Operation</th>
<th id="d1817e504">Study ID Location</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e501">create, update</td>
<td headers="d1817e504">studyid extension element in resource body</td>
</tr>
<tr>
<td headers="d1817e501">search (low volume resources)</td>
<td headers="d1817e504">studyid query string parameter (for example. Device?studyid=STUDY01)</td>
</tr>
<tr>
<td headers="d1817e501">read, vread, history, search (high-volume resources), delete</td>
<td headers="d1817e504">X-WHC-LSF-studyid header</td>
</tr>
</tbody>
</table>

For search requests that conform to the supported API patterns, you must use the studyid URL parameter.

  ```GET [base]/[type]{?[parameters]}```
 ```POST [base]/[type]/_search{?[parameters]}```
    
 In the following scenarios, include the query string parameters that are shown in [Table 5](#wp4h_gxp_c_fhir_details__table_oww_p55_z1b) in FHIR search requests (query string) for study-scoped resources:

<table id="wp4h_gxp_c_fhir_details__table_oww_p55_z1b">
<caption>Table 5. Request parameters for query strings.</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e564">Request parameter</th>
<th id="d1817e567">Scenario</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e564">studyid</td>
<td headers="d1817e567">Required for low-volume, study-scoped resources stored in DB2.</td>
</tr>
<tr>
<td headers="d1817e564">A single "compound key" query string parameter whose value is a concatenation of other fields
(such as. patientIdSiteIdStudyId)</td>
<td headers="d1817e567">Required for high-volume study-scoped resources. For high-volume resources, only one compound
key search parameter can be used per search request. The FHIR server returns a 400 <code>Bad
Request</code>  if you specify more than one compound key in the search request.</td>
</tr>
</tbody>
</table>

### Searching with high-volume, study-scoped resources

High volume resources that are stored in HBase have certain limitations when it comes to the FHIR search operation. Specifically, searches can include only a single compound key search parameter and an optional date/time (`timestamp`) parameter, as shown in the following example:
    
    GET [base]/fhir-server/api/v1/Observation?compoundKey:exact=abc123xyz&date=2017-07-18T00:00:00.000Z

Because compound keys require an exact match, the `exact` modifier is the only legal modifier. Search requests that do not include the exact modifier return a 400 `Bad Request` error. The high-volume data store has a single notion of time stamp for each cell. Therefore, only one time stamp (date/time) can be indexed for each resource type. The time stamp fields that are chosen for high-volume resources are identified in [Table 6](#wp4h_gxp_c_fhir_details__table_f3h_gv5_z1b). Although time stamp (data/time) search parameters are optional, the values for these fields are required for proper indexing, and are validated during ingestion on FHIR create (POST) and FHIR update (PUT) operations.

<table id="wp4h_gxp_c_fhir_details__table_f3h_gv5_z1b">
<caption>Table 6. Time stamps for study-scoped resources</caption><colgroup></colgroup>
<thead style="text-align:center; vertical-align:top">
<tr>
<th id="d1817e639">Resource Type</th>
<th id="d1817e642">Timestamp (date/time) field</th>
<th id="d1817e645">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td headers="d1817e639">MedicationAdministration</td>
<td headers="d1817e642">MedicationAdministration.effectiveTimeDateTime</td>
<td headers="d1817e645">Time of administration.</td>
</tr>
<tr>
<td headers="d1817e639">Observation</td>
<td headers="d1817e642">Observation.effectiveDateTime</td>
<td headers="d1817e645">Clinically relevant time for observation.</td>
</tr>
<tr>
<td headers="d1817e639">QuestionnaireResponse</td>
<td headers="d1817e642">QuestionnaireResponse.authored</td>
<td headers="d1817e645">Authoring date for this version.</td>
</tr>
</tbody>
</table>

> **Note:** Both `MedicationAdministration` and `Observation` have fields ( `MedicationAdminstration.effectiveTimePeriod` and `Observation.efffectivePeriod`) that use the FHIR Period data type. Because the high volume data store has only a single notion of time, you cannot use these fields for searches. Only the fields that are listed in [Table 6](#wp4h_gxp_c_fhir_details__table_f3h_gv5_z1b) can be indexed for search.

## Using the FHIR Registration API (custom operations)
{: #wp4h_gxp_c_fhir_details__section_pyz_wlk_tdb}
The FHIR Registration API provides a set of methods to manage the registration for patients, practitioners, and other people who are involved in a patient's care for clinical studies and commercial programs.

+ For clinical studies, patients and practitioners register directly with the study through the registerPatient or registerPractioner methods.
+ After a patient or practitioner registers for a commercial program, they generally receive an email invitation. To join the commercial program, they need to take the additional step of accepting the invitation, which is managed through the acceptPatientInvitation or acceptCareManagerInvitation methods.
+ For patients who are minors or non-competent adults, the registerThirdParty method allows a guardian (for minors) or a third party (for non-competent adults) to establish a relationship with a patient. The guardian or third party has access to a patient's healthcare data and history.
+ The remaining methods allow administrators to manage patient and practitioners and their relationships:

  - The removeRelationship method removes a third-party or guardianship relationship.
  - The removePatient and removePractitioner methods remove a patient or practitioner from a clinical study or commercial program.

## Additional supported FHIR features

{{site.data.keyword.wh_prodname_short}} also supports the following FHIR features.

* **FHIR Bundles** - Resources can be bundled together (in a single POST request) which allows for efficiencies in terms of batch processing and ingestion of resources. Bundles can optionally be persisted as a single atomic unit of work by using transaction bundles (that is, where Bundle.type = transaction). For more information about bundles, see [https://www.hl7.org/fhir/DSTU2/bundle.html](https://www.hl7.org/fhir/DSTU2/bundle.html).

* **Removing data from the FHIR repository** - The DELETE API call performs a soft delete. That is, when a resource is deleted, the FHIR service creates a new version of the resource, along with a 'deleted' indicator. This indicator does not exist within the resource JSON itself, but is persisted along with the resource to indicate to the persistence layers that the resource is no longer available. For more information about deletion with FHIR resources, see [https://www.hl7.org/fhir/DSTU2/http.html#delete](https://www.hl7.org/fhir/DSTU2/http.html#delete).

