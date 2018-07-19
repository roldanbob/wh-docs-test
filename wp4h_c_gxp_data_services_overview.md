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

# Data services overview

With {{site.data.keyword.wh_prodname_long}} , you can ingest health data from many sources. You can then use that data to help clients create innovative solutions that provide medical insights and help patients take charge of their own health outcomes.

As a user, such as a data scientist, analyst, and other researcher, you want the ability to analyze large data sets by using industry-standard tools. {{site.data.keyword.wh_prodname_short}} supports ingestion of data that comes from many sources, such as electronic health records (EHRs), medical devices, mobile applications, and fitness or wellness devices.

{{site.data.keyword.wh_prodname_short}} provides several APIs and services for ingesting, managing, and analyzing your healthcare data. Data can be ingested either in the form of Fast Healthcare Interoperability Resources (FHIR) or as files in their original format (such as CSV, text, or binary). FHIR data is then normalized, masked, and stored in a data reservoir, where the data is available to you for analysis and reporting. File data is not processed by the {{site.data.keyword.wh_prodname_short}} , but can be stored for convenience in either the data lake or data reservoir.

FHIR is an international standard for sharing data in the life sciences and healthcare spheres. FHIR resources refer to data that is formatted into modular components that can be used as building blocks for applications. {{site.data.keyword.wh_prodname_short}} currently supports FHIR DSTU2. For more information, see the [FHIR DSTU2 Summary ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/summary.html){: new_window}.

Data in the {{site.data.keyword.wh_prodname_short}} can also be stored in a {{site.data.keyword.prodname_pde_warehouse}} for sharing with authorized clinical researchers and case managers outside of your organization. Users can include members of clinical research organizations (CROs) and care management organizations (CMOs). For more information, see the [GxP {{site.data.keyword.prodname_pde_caps}} service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWH8G9qNwiyMWyltT-DL){: new_window} topic.

[Figure 1](#wp4h_c_data_services_overview__fig_ds_flow) shows, at a high level, how FHIR resources flow from customer applications through the FHIR repository, data lake, and data reservoir. The final output is available to analyst users for analysis and reporting by using Spark jobs and Hive queries. In addition, through REST APIs, file data can be added directly to (and retrieved from) the data lake and data reservoir.

*Figure 1. Data Services data flow diagram*

![Data flow for both FHIR resources and file data. Data comes into the FHIR service from a customer application and is sent to the appropriate high- or low-volume database and replicated. The replicator service sends data to the FHIR data lake. The data lake then sends data to the FHIR data reservoir for analysis or to the patient data warehouse for inclusion in clinical studies.](userImages/ai_gxp_ds_data_flow_21.jpg)

As shown in [Figure 1](#wp4h_c_data_services_overview__fig_ds_flow), data from customer applications takes the following path to become available to an analyst user:  {: #wp4h_c_data_services_overview__section_eqh_w3r_p1b}

1. Data from customer applications is ingested into the FHIR data repository through the FHIR REST API. The Data Services section of the diagram includes the following services and applications:

    - [Consent manager ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWDRPqPPya8_GHWo5woD){: new_window} and [Role and Relationship Access Manager ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWIaKn1CwiyMWyltT-FP){: new_window} services - Consent manager tracks whether patients have granted consent for use of their data for an existing study. A patient's data can flow into the FHIR repository only if the study exists and the patient's consent is registered in the Consent database. {{site.data.keyword.prodname_rram_first}} helps ensure that users who access patient data have appropriate roles and authorization.
    - FHIR data repositories - Stores all FHIR data to make it available for other processes. FHIR data can also be retrieved by using the FHIR API.
    - Replicator process - Processes and copies FHIR data to the data lake so it can be used by other data management services, such as {{site.data.keyword.prodname_cro_dw}} ({{site.data.keyword.prodname_adherence}}), {{site.data.keyword.prodname_dle_caps}} ({{site.data.keyword.prodname_dle_short}}), or Data De-Identification:

        - {{site.data.keyword.prodname_cro_dw}} service (described in the [GxP {{site.data.keyword.prodname_pde_caps}} service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWH8G9qNwiyMWyltT-DL){: new_window} topic) - Specified data from the data lake is sent to the {{site.data.keyword.prodname_pde_warehouse}}. From there, use the {{site.data.keyword.prodname_adherence}} API to find, aggregate, and export patient data in the context of clinical studies or commercial programs.
        - {{site.data.keyword.prodname_dle_caps}} ({{site.data.keyword.prodname_dle_short}}) service - Authorized users (such as clinical researchers) can use the {{site.data.keyword.prodname_dle_short}} REST API to export clinical data for a FHIR resource from a specified {{site.data.keyword.study}}.
        - [GxP Data De-Identification ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWIfUP6BwiyMWyltT-F_){: new_window} - From the data lake, data moves to the Data De-identificaton service, which removes or masks protected health information (PHI) to assure privacy for data in the data reservoir. From the data reservoir, the masked data is available for analysis and reporting. The service masks dozens of data fields, and can be configured to meet your business needs. All FHIR data in the data reservoir is masked.

    - Non-FHIR data lake - The data lake and {{site.data.keyword.prodname_dl}} API are available to store and retrieve files in their original formats to support additional business needs.

1. In the Analytics section of the diagram, data in the data reservoir is available to analyst users. FHIR data can be used for analysis and reporting by using Jupyter Notebooks with Spark jobs, Hive queries, and other analytical languages and packages. For more information, see the [Analytics Subsystem service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://rtpdoc01.rtp.raleigh.ibm.com:9443/kc/SSSMS8/content/wp4h_gxp_c_analytics_datamarts.html){: new_window} topic in the IBM Knowledge Center. In addition, the non-FHIR data reservoir and {{site.data.keyword.prodname_dr}} API are available to store masked data files in their original formats. You can run Spark jobs against file data in certain formats (such as CSV, JSON, Parquet, and TXT) from the file data reservoir.

 Because the data reservoir is under HIPAA and GxP control, any data that is used for analysis is expected to contain only masked and anonymous data. When adding files to the data reservoir for analytics, the study designer must ensure that any personal information in files that are uploaded to the data reservoir is removed or masked. Personal information includes any personally identifiable information (PII), protected health information (PHI), or sensitive personal information (SPI). User analysts can use Spark to add data from the file data reservoir to Jupyter Notebook. From the file data reservoir, Spark has been tested with data in CSV, JSON, Parquet, and TXT formats.

The data services overview focuses on three data services; FHIR Ingestion, {{site.data.keyword.prodname_dl}}, and Data Reservoir. However, a number of other tools and services are used in the context of the {{site.data.keyword.wh_prodname_short}} that are mentioned here but discussed in detail in other documentation. You can find additional information elsewhere about each of the following services:

- {{site.data.keyword.prodname_as}}. Tracks all logins and logouts, data movement, and errors to comply with government regulations. Each service has its own audit trail. Audit data is stored in an Audit data store. For more information, see the [Audit Trail Export service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AWIH9hIWwiyMWyltT-Erl){: new_window} topic.
- {{site.data.keyword.prodname_lma_long}} ({{site.data.keyword.prodname_lma_short}}) service. Provides logging, monitoring, and alerting functions for {{site.data.keyword.wh_prodname_short}} solutions. For more information, see the [GxP {{site.data.keyword.prodname_lma_short}} Core service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://watsonpow01.rch.stglabs.ibm.com/services/cognitive_catalog/catalog/asset.html?id=AV5JRCBoSgsFwvqnf1Ky){: new_window} topic.
- Spark jobs, Hive queries, and Jupyter Notebook. Tools available to analyst users for use in analytic studies. For more information about the Analytics Subsystem, see the [Analytics Subsystem service ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSSMS8/content/wp4h_gxp_c_analytics_datamarts.html){: new_window} in the IBM Knowledge Center.
- In addition, High Availability/Disaster Recovery (HADR) services are shown in the Data Services data flow diagram. The HADR services are internal services that are used only by IBM® personnel to help ensure that the {{site.data.keyword.wh_prodname_short}} runs smoothly.
