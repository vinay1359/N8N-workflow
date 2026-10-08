# Generate an Image Meme from a Topic

This workflow lets a creator enter a topic in an n8n form and generate a downloadable image meme. DeepSeek writes and judges the joke, and Imgflip renders it on a suitable template. The workflow can revise a weak meme up to three times. It does not need a separate topic database or file storage account.

## File in this folder

- creator-meme.json - importable n8n workflow

## How the workflow works

1. **Enter meme topic** shows a form with one required Topic field. Each submission starts one execution and receives an immediate acknowledgement.
2. **Config and input** validates the topic and starts the run with attempt 0, an empty best-result slot, and quality settings.
3. **Assemble meme prompt** offers DeepSeek a shortlist of curated two- and three-box Imgflip templates. On retries, it avoids templates already used in this run and includes the previous judge's feedback.
4. **Orchestrator - DeepSeek** writes one joke as structured JSON: template choice, one short line for each box, and a social caption. **Parse meme** checks and normalizes that response.
5. **Build caption request** encodes the template ID and box text. **Caption on Imgflip** renders the image using an n8n Header Auth credential. **Extract meme URL** records the image URL or Imgflip error.
6. **L1 Lint** checks for complete caption text and a rendered image. An incomplete image skips the audience judge and is scored zero.
7. **L4 Audience judge - DeepSeek** scores relatability, humor, template fit, clarity, and shareability from 0 to 100. **Collect scores** records the score and suggested fixes.
8. **Gate and best attempt** keeps each image together with its own text, template, caption, and score. A clean image passes at 80 or higher. Otherwise **Apply revision** loops back for a new candidate. The loop ends after at most three attempts.
9. **Select best meme** chooses the passing image, or the best complete image for review if none passed. **Download selected meme** downloads its binary into the n8n execution. If no attempt produced a valid image, the run fails clearly.

The state of the run travels as one item: topic, attempt count, used template IDs, revision feedback, and best completed candidate. Keeping the full candidate together prevents captions from one attempt being paired with another image.

## Requirements

Before setup, keep these ready:

- A running n8n instance with Form Trigger, Code, HTTP Request, and If nodes
- An Imgflip API key from Imgflip API Settings
- A DeepSeek API key
- Access to the workflow's Executions page to download the result

## Setup steps

1. Import creator-meme.json into n8n.
2. Create a Header Auth credential with header name Authorization and value Bearer YOUR_IMGFLIP_API_KEY. Select it on **Caption on Imgflip**.
3. Create a second Header Auth credential with header name Authorization and value Bearer YOUR_DEEPSEEK_API_KEY. Select it on **Orchestrator - DeepSeek** and **L4 Audience judge - DeepSeek**.
4. Save the workflow. For a test, click Execute workflow and submit the Test URL form. For regular use, publish the workflow and share the Production URL from **Enter meme topic** with trusted users.
5. Submit one test topic, open its execution, inspect the quality decision, and download the image from the Binary tab of **Download selected meme**. Open the file to confirm the text fits the intended boxes.

The DeepSeek requests use the supported deepseek-v4-pro model in non-thinking JSON mode. Credentials are added inside n8n after import; the exported JSON contains no keys or credential IDs.

## Example topic

- The first five minutes of a Monday meeting

The user types this into the form. They do not edit a workflow node for each meme.

## Expected output

The execution ends with one image file on **Download selected meme**. **Select best meme** also shows the topic, Imgflip image URL, score, attempt count, selected template, caption, and approved or review status. The submitter sees an acknowledgement immediately; the finished file is downloaded from the n8n execution because multiple AI attempts may outlast a browser response.

## Notes

- Imgflip-generated image URLs are accessible to anyone who knows the URL and may be removed by Imgflip after low usage. Download the binary promptly.
- n8n retains the binary according to its execution retention settings. No automatic social posting happens.
- The audience score is an AI editorial estimate. A creator should still look at the final image before posting.
- One form submission can make up to three Imgflip renders and multiple DeepSeek calls.

## Troubleshooting

If the form does not open, confirm the workflow is published for its Production URL, or click Execute workflow before using the Test URL.

If an API node fails, confirm its Header Auth credential is attached and the provider account is active. Inspect that node's error in Executions.

If Imgflip returns no image, confirm the API key, template availability, and the encoded caption body on **Build caption request**. Three failed renders end with an error rather than a blank download.

If the meme scores too low, inspect the feedback and the selected template in the execution. The retry loop is capped at three, so it cannot run indefinitely.

## Security reminder

This export contains no provider secrets. Keep API keys in n8n credentials and restrict access to the form and Executions page. A public form can consume paid API calls. Topics, prompts, scores, and provider URLs are stored in n8n execution data even though this workflow uses no separate database or Drive.

## Verification status

The workflow JSON, node links, Code node syntax, request expressions, Imgflip request body, and retry decisions were checked with local mock data. A live provider run still requires your own n8n instance and credentials. The setup test above is the final end-to-end check.
