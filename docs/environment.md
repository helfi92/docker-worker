---
title: Environment Variables
order: 20
---

Environment variables can be provided in the task payload and will be added to the
current environment configuration. Environment variables can be both encrypted or
plain text. Refer to the [Encrypted Environment Variables](#encrypted-environment-variables)
section for more information.

#### Reserved Environment Variables

In addition to any environment (env) variables given we also provide every
docker-worker task with the following environment variables these are mandatory
and override any task provided values.

    - `TASK_ID` : The current task id.
    - `RUN_ID` : The current run id for the task.

Note that environment variables can also be used in the `command` field.

```json
{
  "command": ["/bin/bash", "-c", "curl https://queue.taskcluster.net/v1/task/$TASK_ID"]
}
```

#### Encrypted Environment Variables

**WARNING**: we do not recommend using encrypted environment variables.
Instead, prefer to use the secrets service.  Encrypted environment variables
are currently secure, but if the private key is ever disclosed, *all* formerly
protected values will be readable to anyone posessing that key.  This section
remains here only for reference, since encrypted environment variables are
still in use for some tasks.

Environment variables can be encrypted to allow secure transmission of private information
such as access tokens, passwords, etc. Secure environment variables must be encrypted
using a public key and then base64 encoded prior to submitting the task.

Each encrypted environment variable must include the message version, task ID,
start and end time, and the name and value of the environment variable.

Encrypted variables are validated by inspecting the task ID as well as the start
and end times to prevent stealing/tampering of the secured variables.

Note: Because the task ID and timestamps are used during validation, this prevents
encrypted variables being reused between tasks (e.g. manual job retriggers on treeherder).

In the example below, encrypted(raw-message) is a gpg encrypted object using the
public key located at [references.taskcluster.net](http://references.taskcluster.net/docker-worker/v1/docker-worker-pub.pem).
Encrypted environment variables are then base64 encoded and included under encryptedEnv in the task payload.

Raw message example:

```js
{
  "messageVersion":     1,
  "taskId":             "<taskId>",
  "startTime":          1418146006679, // As number of ms since epoch
  "endTime":            1418146036679, // As number of ms since epoch
  "name":               "SECRET_TOKEN",
  "value":              "<secret-value>"
}
```

Task payload example with encrypted raw message:

```js
{
  "task": {
    [...]
    "payload": {
      [...]
      "encryptedEnv": ["<base64(encrypted(raw-message))>"]
    }
  }
}
```

Once decrypted within docker-worker, the variable can be referenced just like any other environment variable.

```js
{
  "command": ["/bin/bash", "-c", "echo $SECRET_TOKEN"]
}
```
