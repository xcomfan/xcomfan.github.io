---
layout: page
title: "Observability"
permalink: /system_design/observability
---

## Core Analysis loop

Below is the core loop which is the basis of debugging from first principles. You can use this loop as a brute force method to cycle though all available dimensions to identify which ones explains or correlate with the outlier in question. In theory no prior knowledge of the system is required.

For observability domain **rows** pertain to individual telemetry events and **columns** pertain to the fields and attributes of those events.

1. Start with the overall view of what prompted your investigation: what did the customer tell you?
2. Verify that what you know so far is true: is a notable change in performance happening somewhere in this system? Data visualizations can identify changes of behavior (think os a spike of a graph).
3. Search for dimensions that might drive that change in performance.  Approaches to accomplish this might include.

    a. Examining sample rows (events) from the area that shows the change: are there any outliers in the columns (details of event) that might give you a clue?

    b. Slicing those rows across various dimensions looking for patterns: do any of those views highlight distinct behavior across one or more dimensions?  Try an experimental group by status code.

    c. Filtering for particular dimensions or values within those rows to better expose potential outliers.

4. Do you know enough about what might be occurring? If yes you are done if not filter your view to isolate this area of performance as your next starting point and return to step 3.

