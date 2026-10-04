# Health and diagnostics

[Documentation home](Home.md)

```bash
comai doctor
comai doctor --json
```

The doctor checks the **active provider and configured model** with a read-only
model listing. It does not generate an answer. JSON uses `schema_version: 1`;
exit 0 means healthy and exit 1 means a diagnosed failure.

| Status | Next action |
| --- | --- |
| `ok` | The endpoint lists the configured model. |
| `missing_credentials` | Configure the provider's API key. |
| `bad_credentials` | Check credentials and provider access permissions. |
| `missing_model` | Run `comai models`; choose an available model. |
| `timeout` | Check network access and server responsiveness. |
| `unreachable` | Check the endpoint, running server and TLS/network settings. |
| `server_unavailable` | Check the provider/server status and retry later. |
| `rate_limited` | Wait for the provider limit to reset. |
| `invalid_response` | Confirm the endpoint provides the expected model-list API. |
| `invalid_endpoint` | Correct the base URL; credentials in URLs are rejected. |
| `missing_dependency` | Install the required `curl` or `jq` command. |
| `key_command_failed` / `untrusted_key_command` | Review your key command and config ownership/permissions. |
| `endpoint_error` | Check the API route and endpoint configuration. |

Diagnostics omit keys, endpoint URLs and raw server response bodies. Model-list
availability does not guarantee that every generation request will succeed.

Use `comai status` to check connections across providers and `comai config show`
to inspect masked configuration. See [Troubleshooting](Troubleshooting.md) for
server startup and installation problems.
