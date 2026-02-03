# Data Synchronization

The SDK uses websocket to synchronize local flag data with the FeatBit server. Data is stored in memory by default, and updates are pushed whenever flags or related data change.

- Average synchronization time is less than 100 ms.
- If the connection is interrupted (e.g., internet outage), it resumes automatically.
- If you want to use your own data source, use offline mode and bootstrap JSON.

See also: [references/offline-mode.md](references/offline-mode.md)
