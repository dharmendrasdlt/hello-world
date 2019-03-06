# SMS Sending

Code Repository
- [Marketing](https://github.com/DTSL/process_mailin)
- [Transactional](https://github.com/DTSL/mailin_zend)
- [Callback](https://github.com/DTSL/sms-callback)
- [Replies](https://github.coim/DTSL/process-redirection)

## Description

This Application is responsible for sending campaigns/marketing and transactional/notification SMS

## APIs
  * V2
    * [Marketing](https://apidocs.sendinblue.com/mailin-sms/#2)
    * [Transactional](https://apidocs.sendinblue.com/mailin-sms/#1)
  * V3
    * [Marketing](https://developers.sendinblue.com/reference#sms-campaigns-1)
    * [Transactional](https://developers.sendinblue.com/reference#transactional-sms-1)

- All related queues live in the prod-rmq-sendintime rabbitmq cluster (except for `bounce` which lives in `prod-rmq-backend-mta`)
- The two job trigger mechanisms (involving two queues with one message traveling back and forth) shown in the pics above:
  - Ensure that one and only one producer is running at a time
  - Allow high availabilty of the producer service
  - Provide a sort of random load balancing: whichever producer consumes the message gets to run
- At the time of writing, all of the email-sending processes run on the `proc` servers
- `client's email db` refers to clients' databases that live in `cluster7`, `clust-users-1` and `clust-users-2`
- There is a 0 sec TTL index (the field is called `expired`) on the `rs-queues-1/campaign_sending.emailtosend` mongodb collection. At the time of writing, it is used to delete the documents from the collection

## Caveats

- `sendtomta` can be resource-intensive
- `emailprocess consumer` won't publish a message to `Q_in_progress` until all emails have been inserted into `rs-queues-1/campaign_sending.emailtosend`, and that can take a while

## Deployment

Each process has its own Ansible role:

- [email_sending_emailprocess_consumer](https://github.com/DTSL/Deploy_Server_Automation/tree/new_infra/roles/email_sending_emailprocess_consumer)
- [email_sending_emailprocess_producer](https://github.com/DTSL/Deploy_Server_Automation/tree/new_infra/roles/email_sending_emailprocess_producer)
- [email_sending_expired_process](https://github.com/DTSL/Deploy_Server_Automation/tree/new_infra/roles/email_sending_expired_process)
- [email_sending_sendtomta](https://github.com/DTSL/Deploy_Server_Automation/tree/new_infra/roles/email_sending_sendtomta)

Deployment is normally triggered via [Tower](https://tower.51b.tech) jobs

## Skip Process 

If we want to skip any process for a client, we need to update app names in `skip_processing` field of `central_db.users` collection.

Example:
`
    "skip_processing": [
             "email-sending-abtest-winner-consumer",
             "email-sending-contact-analysis-consumer",
             "email-sending-emailprocess-consumer",
             "email-sending-expiredprocess",
             "email-sending-sendtomta",
             "email-sending-testmail-consumer"
    ]
`

If we want to skip whole `email-sending` process we need to update follwoing in `skip_processing` field of `central_db.users` collection.

`
    "skip_processing": [
            "email-sending"
    ]
`

## References

Draw.io diagram sources can be found [here](https://drive.google.com/drive/folders/1HG-5pv5s3WnhK7TRDQ0mDaRGPWHcfoG6?usp=sharing)
