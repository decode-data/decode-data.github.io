# Events
## Overview
The `sessions` table is a flattened session-level aggregation of the raw inbound `events_YYYYMMDD` table. The attibution model defaults to `first_click`, but can also be configured as `last_click`, `first_click_non_direct` or `last_click_non_direct`.

One row represents one single discrete session.

## Schema
The schema design depen
