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

 # Health data services

With {{site.data.keyword.wh_prodname_long}} , you can ingest health data from many sources. You can then use that data to create innovative solutions that provide medical insights, and help patients take charge of their own health outcomes.

As an analyst user, such as a data scientist, analyst, and other researcher, you want the ability to analyze large data sets by using industry-standard tools. {{site.data.keyword.wh_prodname_short}} supports ingestion of data that comes from many sources, such as medical devices, mobile applications, and fitness or wellness devices.

{{site.data.keyword.wh_prodname_short}} provides several APIs and services for ingesting, managing, and analyzing your healthcare data. Data can be ingested either in the form of Fast Healthcare Interoperability Resources (FHIR) or as files in their original format (such as CSV, text, or binary). FHIR data can be normalized, masked, and then stored in a data reservoir, where the data is available to you for analysis and reporting. File data is not processed by the {{site.data.keyword.wh_prodname_short}} , but can be stored for convenience in either the data lake or data reservoir.

FHIR is an international standard for sharing and exchanging data electronically to support the needs of life sciences and healthcare organizations. FHIR resources refer to data that is formatted into modular components that can be used as building blocks for applications. {{site.data.keyword.wh_prodname_short}} currently supports FHIR DSTU2. For more information, see the [FHIR DSTU2 Summary ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://www.hl7.org/fhir/DSTU2/summary.html){: new_window}.

Data in the {{site.data.keyword.wh_prodname_short}} can also be stored in a {{site.data.keyword.prodname_pde_warehouse}} for sharing with authorized clinical researchers and case managers outside of your organization. Users can include members of clinical research organizations (CROs) and care management organizations (CMOs). For more information, see [Patient data for clinical research](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_cro.html).

## Data flow

[Figure 1](#wp4h_gxp_c_data_services__fig_ds_overview) provides a high-level overview of how FHIR resources flow from customer applications through the FHIR repository, data lake, and data reservoir. The final output is available to analyst users for analysis and reporting by using Spark jobs and Hive queries. In addition, through REST APIs, file data can be added directly to (and retrieved from) the data lake and data reservoir.

*Figure 1. Data services overview diagram*

![Data flows from the FHIR repository through the FHIR data lake, to the personal identification protection (masking) service, and then to the FHIR data reservoir. The image has two sections, Data Services and Analytics. In addition, the image shows that data can be stored in (and retrieved from) the non-FHIR data lake or data reservoir.](userImages/ai_kc_data_services_overview_21.jpg) As shown in [Figure 1](#wp4h_gxp_c_data_services__fig_ds_overview), data from customer applications takes the following path to become available to an analyst user:

1. Data from customer applications is ingested into the FHIR data repository through the FHIR REST API. The Data Services section of the diagram includes the following services and applications:

    - [Consent management](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_consent_mgr.html) and [Role and relationship management](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_r_rram.html) services - Consent manager tracks whether patients have granted consent for use of their data for an existing study. A patient's data can flow into the FHIR repository only if the study exists and the patient's consent is registered in the Consent database. {{site.data.keyword.prodname_rram_first}} helps ensure that users who access patient data have appropriate roles and authorization.
    - FHIR data repositories - Store all FHIR data to make it available for other processes. FHIR data can also be retrieved by using the FHIR API.
    - Replicator process - Processes and copies FHIR data to the data lake so it can be used by other data management services, such as {{site.data.keyword.prodname_pde_caps}} ({{site.data.keyword.prodname_adherence}}), {{site.data.keyword.prodname_dle_caps}} ({{site.data.keyword.prodname_dle_short}}), and Personal Information Protection:

        - {{site.data.keyword.prodname_pde_caps}} service (described in [Patient data for clinical research](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_cro.html)) - Specified data from the data lake is sent to the {{site.data.keyword.prodname_pde_warehouse}}. From there, use the {{site.data.keyword.prodname_adherence}} API to find, aggregate, and export patient data in the context of clinical studies or commercial programs.
        - [{{site.data.keyword.prodname_dle_caps}} service](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_data_lake_export_service.html) - Authorized users (such as care managers and clinical researchers), can use the {{site.data.keyword.prodname_dle_short}} REST API to package and export clinical data for a {{site.data.keyword.study}}.
        - [Personal Information Protection](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_personal_information_protection.html) (De-identification) - From the data lake, data moves to the Personal Information Protection service, which removes or masks protected health information (PHI) to assure privacy for data in the data reservoir. From the data reservoir, the masked data is available for analysis and reporting. The service masks dozens of data fields, and can be configured to meet your business needs. All FHIR data in the data reservoir is masked.

    - Non-FHIR data lake - The data lake and {{site.data.keyword.prodname_dl}} API are available to store and retrieve files in their original formats to support additional business needs.

1. In the Analytics section of the diagram, data in the data reservoir is available to analyst users. FHIR data can be used for analysis and reporting by using Jupyter Notebooks with Spark jobs, Hive queries, and other analytical languages and packages. For more information, see the [Analytics subsystem](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_analytics_datamarts.html). In addition, the non-FHIR data reservoir and {{site.data.keyword.prodname_dr}} API are available to store masked data files in their original formats. You can run Spark jobs against file data in certain formats (such as CSV, JSON, Parquet, and TXT) from the file data reservoir.

 ## Related services and utilities

Select APIs and services are available to IBM services teams for building applications on a customer's behalf. In addition, analyst users outside of IBM with authorized access to the Analytics subsystem can use the {{site.data.keyword.prodname_dr}} API to access file data. For more information about using the {{site.data.keyword.prodname_dr}} API from the Analytics subsystem, see [Retrieving files through the {{site.data.keyword.prodname_dr}} API](/docs/services/DATA-SERVICES-FHIR/whac_dsg_c_dr_api.html).

Data services also interact with the following tools and services in the {{site.data.keyword.wh_prodname_short}}.

## Health data services
{: #wp4h_gxp_c_data_services__section_jlh_jsn_5cb}

Health data comprises the following services. For more information about each service, click the service name.

- [**FHIR service**](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_fhir_service.html)<br/> The {{site.data.keyword.wh_prodname_long}} Fast Healthcare Interoperability Resources (FHIR) service and REST API support high-volume ingestion of electronic data from multiple sources, including healthcare devices and mobile applications. After processing, FHIR data is accessible to analysis and reporting tools such as Spark jobs and Hive queries and for use in clinical research.
- [**Data services APIs**](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_kc_data_services_api.html)<br/> {{site.data.keyword.wh_prodname_long}} Data Services includes APIs for FHIR, data lake, and data reservoir that you can use to manage patient data from medical devices and other sources.

**Related concepts**

- [Analytics subsystem](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_analytics_datamarts.html)
- [Audit logging and trails](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_audit_logging_and_trails.html)
- [High availability and disaster recovery](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_disaster_recovery.html)
- [Personal information protection](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_personal_information_protection.html)
- [Platform monitoring](/docs/services/DATA-SERVICES-FHIR/wp4h_gxp_c_platform_monitoring.html)
