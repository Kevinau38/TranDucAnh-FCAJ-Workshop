---
title : "Troubleshooting and best practices"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Common problems

| Problem | What to do |
|---|---|
| `screener: command not found` | The alias is lost in a new terminal or session. Go back to the `service-screener-v2` folder and run `alias screener='python3 $(pwd)/main.py'`, or repeat chapter 2 if `/tmp` was cleared. |
| `CredentialRetrievalError` during a long scan | CloudShell serves credentials through a local proxy that can fail under load. Before the scan, run `eval "$(aws configure export-credentials --format env)"`. These credentials expire with the CloudShell session (usually 1–3 hours). Then re-run the scan and answer `y` to resume. |
| `AccessDenied` on some checks | The user is missing read permissions for that service. Check with `aws sts get-caller-identity` and add `ReadOnlyAccess`. |
| The scan takes too long | Limit the scope with `--regions`, `--services` or `--tags`, or use `--disable-custom-pages 1` (saves about 2–3 minutes). |
| `--beta 1` shows the old report | The Cloudscape UI was not built. Repeat step 5 of chapter 2. |

#### Best practices

- **Scan where you have workloads.** A Region that you do not scan is unassessed, not clean.
- **Run it regularly**, for example after each release or every month, and compare the results, or use Well-Architected milestones.
- **Fix in order**: High severity first, then Quick Wins, then the rest.
- **Suppress what you have accepted** with a suppression file, so that the report shows only new problems.
- **Keep the report private.** Store it on your computer or in a private location, never on a public website.
