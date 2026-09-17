# HumanWeave setup

This is a sanitized snapshot of a 32-node n8n workflow. Import `HumanWeave.json` into a new workflow through n8n's **Import from file** option. It is inactive and requires your own credentials and service settings before it can run. Node names, prompts, positions, and connections are preserved, except for the privacy changes listed below.

## 1. Ollama and Qwen3

Install or use an Ollama service with `qwen3:8b` available. In the **Ollama** node, create or select your own Ollama credential and reachable service URL. A cloud-hosted n8n instance cannot reach a laptop's localhost directly.

## 2. Baserow

Create a Baserow table for the workflow's memory and content records. Create/select a Baserow database-token credential for **Memory**, **Create a row**, **Create a row1**, and **Create a row2**.

Every exported table ID is `0`, deliberately requiring replacement. Select your actual table in each of those four nodes. Remap the field selections in the two nodes that contain explicit field IDs:

| Placeholder field ID | Intended column | Suggested type |
| --- | --- | --- |
| 1 | Topic | Text |
| 2 | Post | Long text |
| 3 | Status | Text, or single select with `Verified` and `Rejected` |
| 4 | AI Generated | Boolean |
| 5 | Person | Text |

These column roles are inferred from the values assigned by the workflow; the original database schema is not included. Confirm field types and remap every field in your own instance. Inspect **Create a row** as well: its original `dataToSend` value is preserved and needs checking against your installed Baserow node version.

## 3. Stable Diffusion WebUI Forge

Run Forge with its API enabled and an image-generation model installed. The export does not specify the checkpoint name; select a suitable model in your Forge installation.

In **HTTP Request4**, replace `https://YOUR_FORGE_HOST.example/sdapi/v1/txt2img` with your own protected endpoint, then select your Basic Auth credential. The original project used Cloudflare Tunnel to connect the local service to n8n Cloud.

The request generates one 512 x 512 image with 20 steps. **Convert to File** converts `images.0` into the binary image used by the upload step.

## 4. Cloudinary

In **HTTP Request5**:

- Replace `YOUR_CLOUDINARY_CLOUD_NAME` in the URL.
- Replace `YOUR_CLOUDINARY_UPLOAD_PRESET` with your configured upload preset.
- Keep the binary input field `data` aligned with the previous node.

This export uses an unsigned upload preset. Apply suitable upload restrictions in your own Cloudinary account. The upload response's `secure_url` is later used in the Buffer request.

## 5. Buffer and LinkedIn

Connect your intended LinkedIn destination in Buffer and obtain API access for your own account.

For **HTTP Request2** and **HTTP Request3**, create/select an **HTTP Header Auth** credential with header name `Authorization` and value `Bearer <your Buffer token>`. Store the actual token in the credential, not in the workflow JSON.

- In **HTTP Request2**, replace `YOUR_BUFFER_ORGANIZATION_ID` in the GraphQL body. This node queries the organization's channels.
- In **HTTP Request3**, replace `YOUR_BUFFER_CHANNEL_ID` with the intended channel ID. The query response does not automatically fill this hardcoded value.

**HTTP Request3 creates a Buffer post with `mode: addToQueue` and `saveToDraft: false`.** After you configure it, running this node can queue a real post for publication. Review your destination, text, and images before running it. For initial testing, change `saveToDraft` to `true` and verify draft behavior with your Buffer account before enabling queue publication.

The original request attaches three image URLs: the portrait, the field-related image, and the generated image. All must be available for this path to work as written.

## Known limitations

- **Portrait fallback is unfinished.** `If2` sends missing portraits to `Search Portrait`, but that node has no outgoing connection. It does not select a verified portrait, populate `imageUrl`, or rejoin the publishing path.
- **Memory field naming is fragile.** `Aggregate` creates a dynamic field with `=Topic {{ $json.Topic }}`, while `Name/Proffesion Generator` reads a literal `Topic Underwater Welding` key. Use a stable field name in both places when adapting the workflow. Also handle an empty Baserow table explicitly.
- **Writing instructions conflict.** `Post creator` contains several different length targets and both an AI-disclosure requirement and a later instruction not to mention AI. Consolidate these prompts before relying on consistent output.
- **Retries have no enforced cap.** `RETRY` increments a counter but the graph has no maximum-retry condition. Add a stopping condition before unattended use.
- **Image selection needs validation.** The field-image expression selects the first result and assumes one exists. Search results need identity, relevance, licensing, and attribution review.
- **AI checks are not independent fact verification.** Review the biography evidence and generated claims. Generated imagery is illustrative and should not be presented as an authentic photograph of the subject.

These issues were documented rather than silently changing the original workflow's behavior. No live execution or Buffer post was triggered during repository preparation.

## What was removed for public sharing

- Hardcoded Buffer authorization values, replaced by credential-based authentication.
- Saved credential names and IDs for Ollama, Baserow, and Forge.
- Buffer organization/channel IDs, Baserow table/field IDs, the private Forge hostname, and Cloudinary cloud/preset settings, replaced by placeholders.
- Pinned sample data and instance-specific workflow metadata.

The public workflow is inactive. Its 32 nodes and connection targets were checked structurally, and the cleaned file was checked for the removed values. Importing and running it with a new set of credentials has not been validated.
