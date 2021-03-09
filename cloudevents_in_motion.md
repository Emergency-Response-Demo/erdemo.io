. https://github.com/cloudevents/spec/blob/v1.0.1/kafka-protocol-binding.md

. https://quarkus.io/blog/kafka-cloud-events/
.. structured binding
+
Allows simple forwarding across multiple protocols. 
However, it may not be efficient and may constraint the type of business data.

.. binary binding
+
Relies on protocol capabilities and enables efficient transfer and encoding. 
If we use the binary mode with Kafka, we will store the data attribute value in the Kafka record’s value and pass the other attributes using the record’s headers. 
Consequently, business data can be encoded using binary protocols such as Avro, leading to higher efficiency.


. https://issues.redhat.com/browse/ERDEMO-92

`````
 Offset: 0   Key: b7a665fb-bad4-46f2-928f-1d227852ccb0   Timestamp: 2021-02-19 11:56:44.439 Headers: ce_datacontenttype: application/json, ce_id: 3b543c89-f3e6-492d-84e4-3ed33a35dba5, ce_source: emergency-response/incident-service, ce_specversion: 1.0, ce_time: 2021-02-19T11:56:44.438272Z, ce_type: IncidentReportedEvent, content-type: application/json
 

{
   "id": "b7a665fb-bad4-46f2-928f-1d227852ccb0",
   "lat": 34.04321,
   "lon": -77.89148,
   "medicalNeeded": false,
   "numberOfPeople": 2,
   "victimName": "Benjamin Cooper",
   "victimPhoneNumber": "(336) 555-5533",
   "status": "REPORTED",
   "timestamp": 1613735804433
}
`````
