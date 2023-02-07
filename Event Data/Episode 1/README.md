**Working with event and alert data**

There are four files provided per episode, containing all the events and alerts generated during the sample execution. There may be events which are unrelated, but are captured as part of the standard event collection.

You can use this data as you see fit, however, the NDJSON files are provided should you wish to load up this data into your own Elasticsearch cluster via Kibana's [file upload feature.](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html#upload-data-kibana)

If you will be uploading to an Elasticsearch cluster via Kibana, please follow these steps.

1. Download the episode's NDJSON file(s). If the file is gzipped, run `gunzip filename.ndjson.gz` to decompress it.
2. Go to Kibana -> Home -> Upload a file

<img width="660" alt="kibanahomescreen" src="https://user-images.githubusercontent.com/20421688/213900298-aa534b8d-16e0-47e5-85a8-1cc1acf4bd37.png">


3. Upload the downloaded NDJSON file
4. Click on Import at the bottom of the page
<img width="566" alt="importbutton" src="https://user-images.githubusercontent.com/20421688/213900289-2e3f4556-3cf2-476f-b930-7dda37176ddb.png">

5. Switch to the Advanced tab
6. Give your index a name starting with `logs-`. This will ensure it will automatically work with Elastic Security. Do not use a pattern of `log-*-*` as we will not be configuring a data stream _**(if you would like to trigger the out of the box rules based on this upload, please follow the additional instructions below)**_.
7. Add an additional index setting - `"mapping.total_fields.limit":"6000"` in the index settings section
8. Replace the contents in the mapping section with the contents of the file `mapping.json` in this repository
<img width="1256" alt="settingapplied" src="https://user-images.githubusercontent.com/20421688/213900279-9da60c77-2e9e-4dfe-8a93-3b58d1b993a4.png">


9. Click on the import button and wait for it to finish. You should see the following if the upload was successful.
<img width="1297" alt="successfulimport" src="https://user-images.githubusercontent.com/20421688/213900262-c6f707da-3a31-43d7-9ea9-2422742958f7.png">

10. You can now view this data in Elastic Security, Discover, or any other area within Kibana. Hint - if you want to filter for this data specifically, use a KQL query that searches specifically for the index you just created, like `_index:"logs-omm_ingested"`
<img width="786" alt="search" src="https://user-images.githubusercontent.com/20421688/213900423-8b3895db-0c31-4bf2-af4e-eb2f82205a11.png">

**Triggering Elastic detections based on the uploaded data**

If you would like to trigger Elastic's out of the box detections based on the uploaded data (for testing/simulation/etc), append the following section to the ingest pipeline during the upload process above:

```
{
      "set" : {
        "field": "event.ingested",
        "value": "{{_ingest.timestamp}}"
      }
    }
```

The full pipeline should now look like this:

```
{
  "description": "Ingest pipeline created by text structure finder",
  "processors": [
    {
      "date": {
        "field": "@timestamp",
        "formats": [
          "ISO8601"
        ],
        "output_format": "yyyy-MM-dd'T'HH:mm:ss.SSSSSSSSSXXX"
      }
    },
    {
      "set" : {
        "field": "event.ingested",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}
```

This will overrite the ingested timestamp from the episode to `now`, which will allow the rules to pick up the events (of course, the rules in question will need to be turned on).

Additionally, you will need to ensure that the uploaded files use these index patterns for the event and alert files respectively (as the rules are set to look for specific index patterns):

`logs-endpoint.events.imported`
`logs-endpoint.alerts-imported`

You can use the same mapping file provided for both.
