# Generate a Video Meme from a Topic

This workflow lets a creator enter a topic in an n8n form and generate a downloadable five-second vertical MP4 meme. DeepSeek plans and judges it, fal.ai makes the motion clip, Creatomate adds readable text, and OpenAI checks a render snapshot. The workflow can revise and render a second candidate if the first fails quality review.

## File in this folder

- creator-video.json - importable n8n workflow

## How the workflow works

1. **Video topic form** asks for one required Topic field. Submission starts one execution and receives an immediate acknowledgement.
2. **Validate topic** rejects blank input. **Prepare video prompt**, **Plan with DeepSeek**, and **Check video plan** produce and validate a short hook, punchline, and a motion prompt that asks for no text in the generated footage.
3. **Prep motion submit** carries the plan and retry state into **Submit motion to fal**. fal.ai starts a five-second 9:16 Kling text-to-video job with native audio disabled.
4. **Wait for motion**, **Poll motion**, **Check motion status**, and **Motion ready?** repeat until the job finishes. The loop checks every 30 seconds, stops on an explicit failure, and times out after 40 unsuccessful polls. **Get motion result** extracts the video URL.
5. **Build video overlay** makes a Creatomate RenderScript: moving background, hook at the start, and punchline later. **Render overlay with Creatomate** starts the 1080 by 1920 MP4 render.
6. **Wait for render**, **Poll render**, **Check render status**, and **Render ready?** repeat every 15 seconds until the overlay finishes. Explicit failure or 40 unsuccessful polls stops the run. The completed render URL and snapshot URL are retained.
7. **L1 Lint** checks that a video and text plan exist. **L3 Visual judge - OpenAI** checks the Creatomate snapshot for clarity, legibility, and topic fit. If the snapshot is unavailable, the visual score is zero. This is a still-frame check; it does not inspect every frame of the MP4.
8. **L4 Audience judge - DeepSeek** scores the hook, punchline, humor, relevance, and shareability. **Gate and best video** passes only when lint is clean, visual score is at least 68, and audience score is at least 66.
9. If the first candidate fails, **Prompt improver - DeepSeek** uses the judge feedback to rewrite the scene and text. **Apply revision** returns to **Prep motion submit** for one more full motion and overlay render. The best complete candidate remains stored in the execution state. There are at most two candidates.
10. **Select best video** chooses the passing MP4 or the best completed MP4 for review. **Download video here** downloads the MP4 into the n8n execution. If no MP4 completed, the run fails.

The run carries one state item with topic, plan, attempt count, current queue and render URLs, poll counters, scores, and best completed candidate. Poll counters reset for a new candidate; the attempt count and best result do not. The best video's URL, plan, and scores stay together.

## Requirements

Before setup, keep these ready:

- A running n8n instance with Form Trigger, Wait, Code, HTTP Request, and If nodes
- DeepSeek, fal.ai, Creatomate, and OpenAI API keys
- Provider accounts with enough available credit for a test render
- Access to the workflow's Executions page to download the MP4

## Setup steps

1. Import creator-video.json into n8n.
2. Create Header Auth credentials in n8n, each using header name Authorization:
   - DeepSeek: Bearer YOUR_DEEPSEEK_API_KEY for **Plan with DeepSeek**, **L4 Audience judge - DeepSeek**, and **Prompt improver - DeepSeek**.
   - fal.ai: Key YOUR_FAL_KEY for **Submit motion to fal**, **Poll motion**, and **Get motion result**.
   - Creatomate: Bearer YOUR_CREATOMATE_API_KEY for **Render overlay with Creatomate** and **Poll render**.
   - OpenAI: Bearer YOUR_OPENAI_API_KEY for **L3 Visual judge - OpenAI**.
3. Save the workflow. Click Execute workflow for its Test URL form. For regular use, publish it and share the Production URL on **Video topic form** with trusted users.
4. Submit one short topic. In Executions, inspect both provider polling loops, the completed render and snapshot, the scores, and the Binary output on **Download video here**. Download and watch the MP4 to check the hook, punchline, timing, and text placement.

DeepSeek uses the supported deepseek-v4-pro model in non-thinking JSON mode. The visual judge uses gpt-4o-mini on a snapshot. The workflow JSON contains no API keys or credential IDs; connect all credentials after import.

## Example topic

- When a meeting could have been an email

The user types this into the form. They do not edit a node for each video.

## Expected output

The final node exposes one MP4 in its Binary tab. **Select best video** also shows the topic, video URL, score, visual and audience scores, attempt count, plan, and approved or review status. The form acknowledges submission immediately. Video creation continues in n8n, so the user downloads from the completed execution instead of waiting for a browser connection to remain open.

## Notes

- A single submission may make two paid fal.ai videos and two Creatomate renders. No automatic posting happens.
- The MP4 is silent. fal.ai native audio is disabled, and no music asset is added.
- OpenAI judges a snapshot only. A human should watch the full video before publishing.
- Provider URLs may expire or have their own retention rules. n8n keeps the binary only as long as its execution retention settings allow. Download a local copy.

## Troubleshooting

If the form does not open, publish the workflow for its Production URL or click Execute workflow before using the Test URL.

If an API request fails, confirm its Header Auth credential, available provider credit, and rate limits. Inspect the failing node in Executions.

If fal.ai returns no queue or result URL, inspect **Submit motion to fal** and **Get motion result**. A FAILED job stops immediately; a job still pending after about 20 minutes times out.

If Creatomate returns no MP4, inspect **Render overlay with Creatomate** and **Poll render**. The render loop stops after about 10 minutes per candidate.

If the snapshot is missing, the visual score is zero. A completed video can still be returned with review status after the second attempt.

If quality is low, inspect visual and audience fixes. The revision loop is capped at two video candidates, so it cannot spend credits indefinitely.

## Security reminder

This export contains no provider secrets. Keep API keys in n8n credentials and restrict form and Executions access. A public form can trigger paid renders. Topics, prompts, scores, and provider URLs remain in n8n execution data even though no separate database or Drive is used.

## Verification status

The workflow JSON, node links, Code node syntax, request expressions, polling state, and quality gates were checked with local mock data. A live provider run still requires your own n8n instance and credentials. The setup test above is the final end-to-end check.
