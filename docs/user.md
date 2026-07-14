# Users Guide

Use Studio to build workflows, run them, and understand what happened when things go right (or wrong).

If you just want to *get something running*, start at **Your first workflow**.

---

## Common tasks

Most people come here to do one of these:

- **Build your first workflow** → [Your first workflow](#your-first-workflow)
- **Use an existing credential in a step** → [Use a credential](#use-a-credential)
- **See why a run failed** → [Debug a failed run](#debug-a-failed-run)
- **Retry a failed step** → [Retry and rerun](#retry-and-rerun)
- **Find files created by a workflow** → [Files](#files)

---

## Studio in one minute

Studio is built around a simple loop:

**Template → Workflow → Instance**

- **Template**: a reusable blueprint (often shared)
- **Workflow**: your configured version (steps + settings + credentials)
- **Instance**: one run of a workflow (results, logs, files, errors)

You don’t need to memorize this. You’ll feel it after your first run.

---

## Your first workflow

Goal: create something small, run it, and watch it finish.

### 1) Create a workflow

Go to **Workflows → Create New** (`/workflows/create`) and choose **From Scratch**.

Give it a name like **My First Workflow**.

### 2) Add one step

Open the editor (`/workflows/{id}/edit`) and click **+ Add Step**.

Pick:
- **Service type** (what kind of thing you want to do)
- **Provider** (who offers it)
- **Service** (the exact action)

**Tip:** start with the **Core** provider if you see it. Core steps are built-in and usually don’t require API keys.

### 3) Configure the step

Fill in the required fields (marked with `*`). Leave everything else alone for now.

### 4) Save and run

Click **Save Workflow**, then **Run**.

You’ll land on the instance view (`/instances/{id}`) where you can watch each step execute.

### What “good” looks like

- The instance moves through steps and finishes as **Completed**
- Each step shows a result payload (even if it’s small)

If you hit **No credentials configured**, that’s normal - it means you chose a provider that needs an API key. Next section.

---

## Credentials and secrets

Studio has two related concepts:

- **Provider credentials**: a login for an external service (API keys, OAuth, etc.)
- **Organization secrets**: reusable values like tokens, URLs, or shared secrets

Both are managed by your org admin. As a user you can browse what already exists at `/secrets` and pick from those values when configuring a step, but you can't create, reveal, or modify them. If a step needs a credential or secret that isn't there yet, ask your admin to add it.

### Use a credential

In a step configuration panel, the credential picker lets you choose from existing provider credentials your admin has set up. Pick one and save the step.

If a step shows **"No credentials configured"**, ask your admin to add a credential for that provider, then run the workflow again.

### Use a secret inside a step

In a step configuration panel, look for places where you can provide a value via mapping.

Common patterns:
- A field lets you select **Secret** / **From secrets**
- Or you use **Input Mappings** and choose a value source

If the secret you need isn't in the picker, ask your admin to create it.

---

## Workflows

Workflows are where you build the automation.

### Where workflows live

- List: `/workflows/list`
- Create: `/workflows/create`
- Edit: `/workflows/{id}/edit`

### Save vs Save As

- **Save** updates the workflow you’re editing
- **Save As** makes a copy (best way to experiment without breaking a working workflow)

### Step modes: Enabled, Skip, Stop

Use these when you’re debugging:

- **Enabled**: runs normally
- **Skip**: bypass the step (data flows through)
- **Stop**: run up to this point, then stop

If your workflow fails halfway through, setting the failing step to **Skip** is often the quickest way to validate the rest of the flow.

---

## How workflows start

Workflows can start in several ways depending on how they're configured. You can pick a trigger type in the workflow editor, but webhook, schedule, API, and event triggers depend on credentials or webhook setup that only an admin can complete.

### Manual

Click **Run** in the workflow editor or from the instance list. The workflow starts immediately with no prompts.

### Form

If any step parameter is mapped to a form input, or if an AI prompt has variables, Studio shows a form before the workflow runs. You fill it out and click Submit (the button label may vary - e.g. "Create Video"). The workflow does not start until you submit the form. This is the same mechanism that powers the Experience View - the form is the trigger.

### Webhook

An external system sends an HTTP request to the workflow's webhook URL. Webhook URL and secret setup is handled by your admin.

### Schedule

The workflow runs automatically on a time interval. The workflow must be set to **Active** for schedule triggers to fire.

### API

The workflow is triggered programmatically via the Studio API. API key setup is handled by your admin.

### Event

The workflow runs when a specific event occurs in a connected app (e.g. a new record in Airtable, a form submission in another service). Provider event hookups depend on credentials your admin manages.

---

## Webhooks

Webhooks let something outside Studio start your workflow. Webhook setup (URL, auth, secret) is handled by your org admin. As a user, your admin will share the webhook URL and secret with you once it's configured for your workflow.

### When a webhook “does nothing”

Check:
- Is the workflow **Active**?
- Did the request reach Studio (you should see an instance created)?
- Is the webhook URL correct (no missing org slug)?
- Is authentication configured correctly (signature/token)?

If the webhook still doesn't fire, ask your admin to verify it's enabled and the secret matches.

---

## Mapping data between steps

This is how workflows become real automation: step outputs feed the next step.

### The idea

Step A produces output (like an image URL). Step B uses that output as input (upload the image).

### Where to map values

In the step configuration panel, find **Input Mappings** (or mapping controls next to fields).

Typical mapping sources:
- **Static value**: type it in
- **From previous step**: choose an output field
- **Form input**: ask the user at run time (if supported)

### Debug mapping problems

If a step fails with “invalid parameters”:
- Confirm required fields are mapped
- Confirm types match (string vs number vs object)
- Check the instance detail to see what the step actually received

---

## Publishing your workflow

You can request to publish a working workflow so others in your org can use it. Publishing is a request - an admin reviews and approves before it's available org-wide.

Before requesting:
- Start from a workflow that runs cleanly end-to-end
- Clean up step names and descriptions
- Make sure no hardcoded sensitive values are baked into step config (use credentials and secrets instead)

Submit the publish request from the workflow editor. Your admin will review and approve or send it back with feedback.

---

## Runs and results

Every time you click **Run**, Studio creates an **Instance**.

### Find your instances

- List: `/instances/list`
- Detail: `/instances/{id}`

### Read an instance like a debugger

When something fails:
1. Open the instance
2. Find the first red step/job
3. Open its error details
4. Check inputs (what the step received)
5. Check outputs from the previous step (what it produced)

### Debug a failed run

Most failures fall into a few buckets:

- **No credentials configured**
  - Ask your admin to add the provider credential, then re-run

- **Credential expired or revoked** (e.g., OAuth token expired, API key rotated)
  - Ask your admin to update or reauthorize the credential
  - Go back to the failed instance and rerun the failed step - no need to start a new run

- **Invalid parameters**
  - A required field is missing or the value type is wrong
  - Fix step config or mappings

- **Timeout**
  - External service is slow or unavailable
  - Try again; if it repeats, reduce workload or check provider status

- **Rate limit exceeded**
  - You hit a provider’s API limits
  - Wait, retry, or reduce concurrency (if your org/super admin configured it)

### Retry and rerun

You don't need to start a new run when something fails. You can fix the problem and pick up where you left off.

You'll typically see actions like:

- **Retry job**: reruns just the failed step
- **Rerun only**: reruns a step (even if completed)
- **Rerun and continue**: reruns a step and downstream steps

Use **Retry job** for flaky network/provider issues.
Use **Rerun and continue** after you change configuration or credentials.

### Recover from a credential failure

This is a common scenario: a step fails because an OAuth token expired or an API key was revoked.

1. Note which step failed and the error (usually an auth/credential error)
2. Ask your admin to reauthorize or update the credential
3. Go back to the failed instance
4. Click **Rerun** on the failed step

The step runs with the updated credential. All previous steps keep their results - nothing is lost or re-executed.

### Delete an instance

You can browse and run instances across the org, but you can only delete the ones you created. Only finished instances (completed, failed, or cancelled) can be deleted.

1. Go to **Instances** (`/instances/list`)
2. Find an instance you created
3. Click **Delete**
4. Confirm

**What gets removed:**
- All output files (images, video, audio, documents)
- All thumbnails
- All database records (steps, jobs, resources)
- The instance directory on disk

This is permanent and cannot be undone. If you need to keep any files, download them before deleting.

If your admin needs to retain an instance for audit purposes, deletion may be disabled - ask your admin if you can't remove one.

---

## Human-in-the-loop

Some workflows pause mid-run waiting for a human action before continuing.

### What you'll see

The instance will show a paused or waiting state. The instance detail page will display a prompt or decision UI - for example, an image selection, a text confirmation, or an approve/reject choice.

### What you need to do

Respond to the prompt to resume the workflow. If you don't respond, the workflow will remain paused until the timeout (if configured) or until you act.

### Patterns

There are two core human-in-the-loop patterns:

- **Approval** - the workflow pauses and asks you to approve or reject before it continues. Common for review gates (e.g. reviewing generated images before they're used in the next step).
- **Notify** - the workflow sends a notification that requires acknowledgment. You confirm you've seen it and the workflow moves on.

### Inside the Experience View

Human-in-the-loop steps also appear inside the Experience View as a "decisions" phase. You may see image pickers, confirmation prompts, or approval dialogs as part of an otherwise seamless experience.

### Troubleshooting

If a workflow seems stuck, check the instance detail for a pending human action before assuming it failed. A paused instance is waiting for you, not broken.

---

## Files

Workflows can generate or download files, and Studio keeps them in one place.

- Files page: `/files`

### What you can do there

- Browse by type (images, video, documents, etc.)
- Preview supported formats (image/video/audio/PDF)
- Download outputs from runs
- Delete files for instances you created or files you uploaded yourself

### If files aren’t showing up

Check:
- Did the workflow step actually create/download a file?
- Did the instance complete that step successfully?
- Are you in the right organization?

---

## Notifications

Notifications help you keep track of what changed and what failed.

- Notifications: `/notifications`

Typical categories:
- workflow events
- job/step failures
- system events
- security/billing (if enabled)

Use notifications for “what happened” and instances for “why it happened.”

---

## Quick reference

### Workflow statuses

- **Draft**: still building (often can run manually, but triggers may be off)
- **Active**: ready to run; triggers enabled
- **Inactive**: paused; won’t respond to triggers
- **Archived**: hidden/retained for records

### Instance statuses

- **PENDING**: created, waiting to execute
- **PROCESSING**: running now
- **WAITING_FOR_WEBHOOK**: paused for external callback
- **PAUSED**: manually paused
- **COMPLETED**: finished successfully
- **FAILED**: error occurred
- **CANCELLED**: stopped by user/system

### Keyboard shortcuts

Flow editor:
- `Ctrl/Cmd + Z` undo
- `Ctrl/Cmd + Shift + Z` redo
- `Delete/Backspace` delete selection
- `Escape` exit fullscreen

General:
- `Ctrl/Cmd + S` save (in editor)