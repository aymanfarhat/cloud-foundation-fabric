## Example build run

Below is an example for triggering the Dataflow flex template build pipeline defined in `cloudbuild.yaml`. The Terraform output provides an example as well filled with the parameters values based on the generated resources in the data platform.

```
GCP_PROJECT="ff-orc"
TEMPLATE_IMAGE="[REGION].pkg.dev/[ORCHESTRATION-PROJECT]/[REPOSITORY]/csv2bq:latest"
TEMPLATE_PATH="gs://[DATAFLOW-TEMPLATE-BUCKEt]/csv2bq.json"
STAGIN_PATH="gs://[ORCHESTRATION-STAGING-BUCKET]/build"
LOG_PATH="gs://[ORCHESTRATION-LOGS-BUCKET]/logs"
REGION="[YOUR-REGION]"

gcloud builds submit \
           --config=cloudbuild.yaml \
           --project=$GCP_PROJECT \
           --region="europe-west1" \
           --gcs-log-dir=$LOG_PATH \
           --gcs-source-staging-dir=$STAGIN_PATH \
           --substitutions=_TEMPLATE_IMAGE=$TEMPLATE_IMAGE,_TEMPLATE_PATH=$TEMPLATE_PATH,_DOCKER_DIR="."
```

## Example Dataflow pipeline launch in bash (from flex template)

Below is an example of launching a dataflow pipeline manually, based on the built template, in the data platform, the launch of this pipeline is handled by an airflow operator.

```
#!/bin/bash

PROJECT_ID=[LOAD-PROJECT]
REGION=europe-west1
SERVICE_ACCOUNT=[LOAD-DF-SA]0@ff-lod.iam.gserviceaccount.com

PIPELINE_STAGIN_PATH="gs://[LOAD-STAGING-BUCKET]/build"
CSV_FILE=gs://[DROP-ZONE-BUCKET]/customers.csv
JSON_SCHEMA=gs://[ORCHESTRATION-BUCKET]/customers_schema.json
OUTPUT_TABLE=[DESTINATION-PROJ].[DESTINATION-DATASET].customers
TEMPLATE_PATH=gs://[ORCHESTRATION-DF-GCS]/csv2bq.json


gcloud dataflow flex-template run "csv2bq-`date +%Y%m%d-%H%M%S`" \
    --template-file-gcs-location $TEMPLATE_PATH \
    --parameters temp_location="$PIPELINE_STAGIN_PATH/tmp" \
    --parameters staging_location="$PIPELINE_STAGIN_PATH/stage" \
    --parameters csv_file=$CSV_FILE \
    --parameters json_schema=$JSON_SCHEMA\
    --parameters output_table=$OUTPUT_TABLE \
    --region $REGION \
    --project $PROJECT_ID \
    --subnetwork="regions/europe-west1/subnetworks/default" \
    --service-account-email=$SERVICE_ACCOUNT
```