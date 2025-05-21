---
title: Data File
layout: home
parent: The Solution
nav_order: 1.4
---

# Data File

The **data file** contains all the health insurance data provided by the health insurers. You can find example data files in the `docs/examples` directory.

## Uploading the Data File

To upload a data file:

1. Open Postman and select the `/Data` request.
2. In the **Auth** tab, choose **API Key** authentication and enter your API Key.
3. In the **Body** section, upload your data file.
4. Click **Send**.

{: .highlight }
Uploading the data file may take several seconds to complete, depending on the file size. It could also fail sometimes, if it fails just retry.

Once uploaded, the system will process and store the health insurance data for use by the application.

## Next step
The next step is to impersonate a doctor and query the data. [Search the data](Search.md)