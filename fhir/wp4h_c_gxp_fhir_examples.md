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

 # FHIR service examples
The FHIR service and FHIR REST API are the heart of {{site.data.keyword.wh_prodname_long}}. Use the FHIR service to add your data to the {{site.data.keyword.wh_prodname_short}} , prepare it, and make it available for scientific analysis in the data mart or for other uses.

The {{site.data.keyword.wh_prodname_short}} is designed for flexibility. The following use cases can help you understand how you can use the {{site.data.keyword.wh_prodname_short}} FHIR data service to upload and search FHIR data.

**Note:** The examples in this section contain generic URL endpoints, user names, and other parameters. When you create your own APIs, be sure to replace any generic text with the correct parameters for your site.

## Designing an asthma inhaler study
<!-- {: #concept_d5b_djm_p1b__section_gfp_qpt_51b} -->
A set of clinicians and research scientists are creating a study for patients with asthma. The patients for this study use an inhaler that is a connected device. Each time a patient takes a puff, the inhaler produces a rich set of information that includes (but not limited to) the following data:

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

During the study design phase, designers work with {{site.data.keyword.deptname_whc}}, data scientists, and other personnel to determine what data is needed for a study. Then, <!-- depending on the data source, -->you can use the FHIR server REST API<!-- , the {{site.data.keyword.prodname_dl}} REST API, or the {{site.data.keyword.prodname_dr}} REST API--> to upload and manage the data.

## Adding FHIR data to the FHIR repository
{: #concept_d5b_djm_p1b__fhir_usecase}

After you design the study, you know which FHIR resources are needed. Use the FHIR Service APIs to add the resources to the FHIR server.

For example, as part of your study, the patient generates data at least daily the following FHIR
resources:

- MedicationAdministration
- Observation
- QuestionnaireResponse

<!--- For an example of uploading the MedicationAdministration FHIR resource to the FHIR repository, see the [FHIR POST example ](https://ibm.box.com/s/31p6aj3z42ror4k8om6jeukserl52a5k){: new_window}. --->

The following sample POST command uploads the MedicationAdministration FHIR resource to the FHIR repository:

```
curl -X POST \
  https://192.168.99.100:9443/fhir-server/api/v1/MedicationAdministration \
  --key <client key> --cert <client cert> \ 
  -H 'Content-Type: application/json' \
  -H 'IBM-App-User: PatientUser-1' \
  -H 'IBM-App-User-IntId: 11111111-1111-1111-1111-1111111111111111' \
  -H 'IBM-DP-correlationid: IBM-DP-correlationid' \
  --data '{
    "resourceType" : "MedicationAdministration",
    "meta" : {
        "versionId" : "3",
        "lastUpdated" : "2017-10-25T06:23:39.901Z"
		},
        "extension": [
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/studyid",
            "valueString": "study001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/siteid",
            "valueString": "site001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/patientid",
            "valueString": "11111111-1111-1111-1111-1111111111111111"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/doseID",
            "valueString": "9"
          },
            {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDsiteIDstdID",
            "valueString": "11111111-1111-1111-1111-1111111111111111_site001_study001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDstdID",
            "valueString": "11111111-1111-1111-1111-1111111111111111_study001"
          }
        ],
        "identifier": [
          {
            "system": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/eventID",
            "value": "36ee9f16-30d9-4d0b-bda6-64823405eae1_1"
          }
        ],
        "status": "completed",
        "patient": {
          "reference": "Patient/f5634e29-963d-4d2e-b192-fa52995af40f",
          "display": "IBMTest_COM_102@us.ibm.com"
        },
        "wasNotGiven": false,
        "reasonGiven": [
          {
            "coding": [
              {
                "system": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/reasonGiven",
                "code": "1"
              }
            ]
          }
        ],
    "effectiveTimeDateTime": "2017-08-08T14:36:33.123Z",
        "medicationReference": {
          "reference": "Medication/d1617804-46f8-4806-b816-7fc59b3dcc26",
          "display": "Rescue"
        },
        "device": [
          {
            "reference": "Device/bbaaa6f3-7ebf-455f-b040-79dd5870666e",
            "display": "Mobile Device"
          }
        ],
        "dosage": {
          "quantity": {
            "value": 100,
            "unit": "ml",
            "system": "http://unitofmeasure.org",
            "code": "mL"
          }
        }
    }
```

For this example, keep in mind that the MedicationAdministration resource includes a number of extensions to the standard MedicationAdministration resource described in the FHIR DSTU2 specification. For the sake of simplicity, only required extensions are shown. For more information about FHIR resources, see the [FHIR DSTU2 Resource list](http://hl7.org/fhir/DSTU2/resourcelist.html){: new_window}.

For all FHIR examples, the `client key` and `client cert` are the key and certificate files of the client application that is signed and issued by the {{site.data.keyword.prodgroupname_short}} certificate authority.

## Adding a FHIR Bundle to the FHIR repository
{: #concept_d5b_djm_p1b__section_pfy_r34_jbb}

You can include multiple FHIR resources into a single transaction that is called a "Bundle."  
The following sample POST command uploads the MedicicationAdministration, Observation, and QuestionnaireResponse FHIR resources as a bundle:
<!--- For example, to include the MedicicationAdministration, Observation, and QuestionnaireResponse FHIR resources as a bundle, see the [FHIR POST Bundle example](https://ibm.box.com/s/y1h3odopp6r8x3jepl5a6vl5pdhq3b6a){: new_window}. --->


```
curl -X POST \
  https://watson-health.example.com/fhir-server/api/v1/MedicationAdministration \
   --key "${HOSTNAME}.nopass.key.pem" \
   --cert "${HOSTNAME}.cert.pem" \   
   -H 'Content-Type: application/json' \
   -H 'IBM-App-User: PatientUser-1' \
   -H 'IBM-App-User-IntId: 11111111-1111-1111-1111-1111111111111111' \
   -H 'IBM-DP-correlationid: IBM-DP-correlationid' \
  --data '{
    "resourceType" : "MedicationAdministration",
    "meta" : {
        "versionId" : "3",
        "lastUpdated" : "2017-10-25T06:23:39.901Z"
		},
        "extension": [
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/studyid",
            "valueString": "study001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/siteid",
            "valueString": "site001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/1.0/patientid",
            "valueString": "11111111-1111-1111-1111-1111111111111111"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/doseID",
            "valueString": "9"
          },
            {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDsiteIDstdID",
            "valueString": "11111111-1111-1111-1111-1111111111111111_site001_study001"
          },
          {
            "url": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/ck/ptnIDstdID",
            "valueString": "11111111-1111-1111-1111-1111111111111111_study001"
          }
        ],
        "identifier": [
          {
            "system": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/eventID",
            "value": "36ee9f16-30d9-4d0b-bda6-64823405eae1_1"
          }
        ],
        "status": "completed",
        "patient": {
          "reference": "Patient/f5634e29-963d-4d2e-b192-fa52995af40f",
          "display": "IBMTest_COM_102@us.ibm.com"
        },
        "wasNotGiven": false,
        "reasonGiven": [
          {
            "coding": [
              {
                "system": "http://www.ibm.com/watsonhealth/fhir/extensions/whc-lsf/r1/reasonGiven",
                "code": "1"
              }
            ]
          }
        ],
    "effectiveTimeDateTime": "2017-08-08T14:36:33.123Z",
        "medicationReference": {
          "reference": "Medication/d1617804-46f8-4806-b816-7fc59b3dcc26",
          "display": "Rescue"
        },
        "device": [
          {
            "reference": "Device/bbaaa6f3-7ebf-455f-b040-79dd5870666e",
            "display": "Mobile Device"
          }
        ],
        "dosage": {
          "quantity": {
            "value": 100,
            "unit": "ml",
            "system": "http://unitofmeasure.org",
            "code": "mL"
          }
        }
    }
```

## Deleting FHIR data
{: #concept_d5b_djm_p1b__d71e145}

After you add a FHIR resource to the FHIR repository, you cannot delete it. However, the DELETE API provides a "soft delete," where the DELETE API call creates a new version of the resource, along with a 'deleted' indicator.

**Note:** When you are deleting a file, use the <code>IBM-WH-reasonforchange</code> header to add information to the audit trail.

For more information, see the FHIR specification delete documentation at [https://www.hl7.org/fhir/DSTU2/http.html#delete](https://www.hl7.org/fhir/DSTU2/http.html#delete){: new_window}.

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

## Searching FHIR data
{: #concept_d5b_djm_p1b__section_g3q_xqm_z1b}

As described in the FHIR specification, you can use the FHIR GET API to find and return records based on criteria that you provide. For more information about searching, see the FHIR specification search documentation at [https://www.hl7.org/fhir/DSTU2/search.html](https://www.hl7.org/fhir/DSTU2/search.html){: new_window}.

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
