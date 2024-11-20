# Guide: Setting Up Elasticsearch Snapshots and Snapshot Lifecycle Management (SLM)

This guide explains step-by-step how to configure and manage Elasticsearch snapshots and automate the process using Snapshot Lifecycle Management (SLM).

---

## 1. **Set Up Elasticsearch Configuration**

### Modify the `elasticsearch.yml` File
Locate the `elasticsearch.yml` file (found in the `config` folder of your Elasticsearch installation) and update the following settings:

```yaml
path.data: C:\Users\Jay Patel\Desktop\elastic-snap
path.logs: C:\Users\Jay Patel\Desktop\elastic-snap\log
path.repo: ["C:\\Users\\Jay Patel\\Desktop\\elastic-snap"]
```

- **`path.repo`**: Specifies the repository location for snapshots.  
Ensure this folder is accessible by Elasticsearch.

### Disable Security Settings (For Local Setup Only)
Add these lines to disable unnecessary security features:

```yaml
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
```

### Save and Restart Elasticsearch
Restart the Elasticsearch service for the changes to take effect.

---

## 2. **Create a Snapshot Repository**

Run the following **cURL command** to create a snapshot repository:

```bash
curl --location --request PUT 'http://localhost:9200/_snapshot/my_snap' \
--header 'Content-Type: application/json' \
--data '{
  "type": "fs",
  "settings": {
    "location": "C:\\Users\\Jay Patel\\Desktop\\elastic-snap",
    "compress": true
  }
}'
```

- **`my_snap`**: Name of the snapshot repository.
- **`type: fs`**: Specifies a file system repository.
- **`location`**: Path to the folder where snapshots will be stored.
- **`compress`**: Enables snapshot compression to save space.

**Expected Response:**
```json
{
  "acknowledged": true
}
```

---

## 3. **Verify the Snapshot Repository**

Run this command to verify the repository:

```bash
curl --location 'http://localhost:9200/_snapshot'
```

**Expected Response:**
```json
{
  "my_snap": {
    "type": "fs",
    "settings": {
      "location": "C:\\Users\\Jay Patel\\Desktop\\elastic-snap",
      "compress": "true"
    }
  }
}
```

---

## 4. **Set Up Snapshot Lifecycle Management (SLM)**

Snapshot Lifecycle Management (SLM) automates snapshot creation and deletion.

### Create an SLM Policy
Run the following command to set up an SLM policy:

```bash
curl --location --request PUT 'http://localhost:9200/_slm/policy/every-5-minutes' \
--header 'Content-Type: application/json' \
--data '{
  "schedule": "0 */15 * * * ?", 
  "name": "snapshot-{now}", 
  "repository": "my_snap",
  "config": {
    "indices": "_all", 
    "ignore_unavailable": false,
    "include_global_state": true
  },
  "retention": {
    "expire_after": "7d", 
    "min_count": 5,
    "max_count": 50
  }
}'
```

### Policy Configuration
- **`schedule`**: Runs every 15 minutes (`0 */15 * * * ?`).
- **`name`**: Snapshot names will be in the format `snapshot-<timestamp>`.
- **`indices`**: Includes all indices in the snapshot.
- **`retention`**:
  - **`expire_after: "7d"`**: Retains snapshots for 7 days.
  - **`min_count: 5`**: Keeps at least 5 snapshots.
  - **`max_count: 50`**: Keeps a maximum of 50 snapshots.

**Expected Response:**
```json
{
  "acknowledged": true
}
```

---

## 5. **Verify the SLM Policy**

Run this command to check if the policy was created:

```bash
curl --location 'http://localhost:9200/_slm/policy/every-5-minutes'
```

**Expected Response:**
```json
{
  "every-5-minutes": {
    "version": 1,
    "policy": {
      "name": "snapshot-{now}",
      "schedule": "0 */15 * * * ?",
      "repository": "my_snap",
      "config": {
        "indices": "_all",
        "ignore_unavailable": false,
        "include_global_state": true
      },
      "retention": {
        "expire_after": "7d",
        "min_count": 5,
        "max_count": 50
      }
    }
  }
}
```

---

## 6. **List Snapshots Manually**

To list all snapshots in the `my_snap` repository, use this command:

```bash
curl --location 'http://localhost:9200/_snapshot/my_snap/_all'
```

**Expected Response (Example):**
```json
{
  "snapshots": [
    {
      "snapshot": "snapshot-2024-11-20T00:00:00.000Z",
      "uuid": "some-uuid",
      "version_id": 8000099,
      "version": "8.0.0",
      "indices": ["index_1", "index_2"],
      "state": "SUCCESS",
      "start_time": "2024-11-20T00:00:00.000Z",
      "end_time": "2024-11-20T00:01:00.000Z",
      "duration_in_millis": 60000
    }
  ]
}
```

---

## 7. **Troubleshooting Tips**

### Repository Permissions
- Ensure the snapshot folder (`C:\Users\Jay Patel\Desktop\elastic-snap`) is accessible by the Elasticsearch process.  
- On Windows, set proper user permissions.

### Restart Elasticsearch
Always restart the Elasticsearch service after modifying the `elasticsearch.yml` file.

### Logs for Debugging
Check the `path.logs` directory for detailed logs in case of errors.

### Security Settings
If using `xpack.security`, ensure SSL certificates are correctly configured.

---

This guide ensures you can set up and manage Elasticsearch snapshots effectively. If you encounter issues, feel free to ask for further help!
