# OpenEMR Standard

## Introduction

OpenEMR is a 2014 ONC Complete EHR Certified medical practice management solution. OpenEMR Standard is a HIPAA-eligible deployment for the Amazon Marketplace &mdash; consult the [datasheet](http://www.open-emr.org/wiki/index.php/OpenEMR_Cloud_Standard_Data_Sheet) for further details.

## Administration

* Access the OpenEMR container: `sudo docker exec -it $(docker ps | grep _openemr | cut -f 1 -d " ") /bin/sh`
* Visit container volume: `docker volume ls`, `cd $(docker volume inspect <volume_name> | jq -r ".[0].Mountpoint")`

### Backups

Your RDS instance will take regular automated snapshots at the configured time of day, and encrypted backups of the OpenEMR filesystem and patient records are made via Duplicity to a stack-provided KMS-managed S3 bucket.

### Restore Procedure

Two restore procedures exist, an automated process using a CloudFormation recovery stack, and a more manual process requiring an administrator put the pieces together.

#### Automated Recovery

Download the automated [stack recovery template](cfn/OpenEMR-Standard-Recovery.json) and create a stack with it in AWS CloudFormation. You'll be answered many of the same questions you were asked when you created your original OpenEMR Standard deployment, but the third section has three new questions.
* Your *KMS key*'s Amazon Resource Name is available from the IAM control panel, under *Encryption Keys*. Remember that an ARN starts with `arn:aws...`. If you can't identify the key used by the stack you want to recover, check the *Resources* tab in CloudFormation for the *OpenEMRKey* entry.
* Your *RDS Snapshot* is the copy of the database you want to restore. Find it in the RDS control panel, under *Snapshots*. This is an ARN, not a name, so click the snapshot to bring up the details.
* You *S3 Bucket* is the bucket created for your backups and audit trail. You can find that under CloudFormation's *Resources* too. Do not provide an ARN here, only the name of the bucket.

With all the questions answered, click through to launch the recovery stack. Once the process has ended, your OpenEMR system will be operating in an entirely new VPC with newly allocated resources, without disturbing the original installation. This faculty can be used to recovery from disaster (reassigning DNS to the new adresses as necessary), to migrate an instance between regions, or just as a monthly scheduled test of the recovery procedure.  

#### Manual Recovery

* Before proceeding, we recommend you make sure to have full copies of all of the following:
  * A full copy of the S3 bucket created by the stack.
  * A full copy of *both* EBS volumes attached to the OpenEMR instance.
  * A snapshot of the production database you are about to overwrite.
* Restore from the most recent Duplicity backup with `sudo /root/restore.sh`.
* Restore the most recent snapshot via the AWS console &mdash; make sure to restore the snapshot into the correct VPC and subnet &mdash; and update your OpenEMR configuration file (`globals.php`) with the endpoint of the new instance.

## Legal Note

[End User License Agreement](https://github.com/openemr/openemr-devops/tree/master/stacks/AWS-mktplace/EULA.txt)
