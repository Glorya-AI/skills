---
name: glorya
description: Make on-brand image ads, video ads, landing pages and websites, and launch ad campaigns, with the Glorya connector. Use it whenever the user asks for an ad, a video ad, a landing page, a website or an ad campaign for their brand, or mentions Glorya.
---

# Glorya

Glorya is an AI marketing agent. It makes ads and pages and launches ad campaigns, and it already knows the user's brand: logo, colors, fonts, products and language. You start the work, follow it and share the results. Glorya finishes on its own unless the user asks to review.

Every tool, input, status, error and limit is in [references/tools.md](references/tools.md).

## When to use

- The user asks for an image ad, a video ad, a landing page, a website or an ad campaign.
- The user wants to change, publish or stop something Glorya made.
- The user asks about their recent Glorya images and videos (`list_assets`), their landing pages and websites (`list_sites`), their campaigns (`list_campaigns`), the video styles (`list_video_formats`) or their brand and products (`get_brand`).
- The user asks about their credits or plan (`get_credits`), or which ad platforms are connected (`get_integrations`).

## Connect

The tools come from the Glorya MCP server at `https://platform.glorya.ai/mcp` (streamable HTTP, OAuth with the user's Glorya account). If they are missing, ask the user to connect it: https://glorya.ai/docs/mcp/quickstart. When they connect, they pick a default brand.

## Rules

- Use the default brand: leave `workspace` empty. Only when the user names another brand, call `list_workspaces` with `query` set to that name and pass the matching `id` as `workspace`.
- Start right away. Pass the user's own words as `brief`. If they gave no details, pass what they asked for, such as "Create an image ad".
- Ask nothing before you start: not the format, size, language, audience or style. Glorya picks these.
- Set `language`, `format` or `references` only when the user gave them. `references` are public https links.
- Leave `review` out; it is false, so Glorya finishes on its own. Set `review: true` only when the user asks to review or approve steps ("let me review", "show me the concept first"). Never ask the user about it.
- Never invent products, prices or offers. Never start the same job twice.

## The job loop

1. Call the create tool. It returns `job_id` and `open_url`. Tell the user it started and share `open_url`.
2. Call `get_job` with `job_id` and `wait_seconds: 30`. While `status` is `running`, call it again. It already waits for changes, so don't poll faster.
3. `needs_input`: Glorya needs a decision. Show each item in `questions` as written, with its exact `choices` and `preview_images`. Wait for the user. Then call `respond` with `job_id`, the question's `request_id` and `answer`, and go back to step 2.
   - `decision` and `approve`: `answer` is one exact label from `choices`.
   - `input`, `revise` and `clarify`: the user's own words work.
   - Don't answer for the user. Don't add questions of your own. If the user writes in another language, translate the question but send the exact label.
4. `completed`: share the results (see each type below) and `open_url`. If `progress` asks the user something, relay it and send the answer with `refine`.
5. `failed`: tell the user what `error` or `progress` says. Offer `refine` with a changed instruction, or a new job.
6. `cancelled`: the job stopped. Finished files stay in Glorya.

Changes: call `refine` with the same `job_id` and the user's words as `instruction`, then run the loop again from step 2. Changes use credits.

## Image ad

- Start with `create_image_ad`.
- Glorya plans the ad, writes the copy and makes the images. With `review: true` it stops to approve the plan (concept and copy) and the finished images.
- When it's done, `assets` holds the images. Share each `url` and `open_url`. `progress` holds Glorya's latest message, such as the ad copy.
- For changes, use `refine`: "Make the headline shorter."

## Video ad

- Start with `create_video_ad`.
- Set `format` only if the user named a video style. Call `list_video_formats` and pass the matching `id`. Otherwise leave it out; Glorya picks one.
- Glorya writes the script, makes the start frames and renders the final cut. With `review: true` it stops to approve the format, the script, the start frames and the final cut.
- Rendering takes minutes. Keep calling `get_job`, and tell the user now and then that it's still working.
- When it's done, `assets` lists the video first, with `url` and `poster_url`. Share the video link and `open_url`.
- For changes, use `refine`: "Use a calmer voice." To stop it, use `cancel_job`.

## Landing page

- Start with `create_landing_page`.
- The builder may ask one clarifying question (`clarify`). Relay it like any other.
- When it's done, share `site.preview_url` and `open_url`. The page is not live yet.
- For changes, use `refine`: "Add a pricing section."
- Publish only when the user says so ("publish it", "make it live"): call `publish_site` with `job_id`. `slug` is optional (lowercase letters, digits and dashes).
- After publishing, call `get_job` until `site.live_url` is set, then share it. If `site.publish_status` shows a failure, tell the user; `publish_site` can run again.
- A build can't be cancelled once it starts.

## Website

- Start with `create_website`. It builds a site with several pages.
- Everything else works like the landing page: one possible clarifying question, `site.preview_url`, `refine` for changes, `publish_site` only when the user asks, then `site.live_url`.

## Campaign

- Start right away with `create_campaign` and the user's own words as `brief`.
- It builds the campaign, makes the ads and launches it with real ad spend, on its own. The ad platform bills the spend to the user's ad account.
- Set `review: true` only when the user asks to review ("let me review the campaign first"). Glorya then keeps the campaign paused until the user approves the launch.
- Don't ask for the platform, goal, budget or destination. Set `platform`, `goal`, `daily_budget`, `destination` or `website_url` only when the user gave them. The default budget is 30 a day (TikTok: 50) in the ad account's currency.
- `integration_required`: the ad platform isn't connected. Share `connect_url`, then try again once the user has connected it.
- When it's done, share `campaign.manage_url` and `open_url`, and call `list_campaigns` for its live status.
- `cancel_job` stops the run, but a campaign that's already live keeps running. The user pauses it in Glorya or the ad platform.
- For how campaigns are doing, call `list_campaigns`.

## Credits

- `get_credits` shows the brand's plan and credit balance. It works even without a plan.
- `no_credits`: share `billing_url` so the user can top up.

## When something fails

Tool errors come with a message, a code and sometimes a link. Pass the message and link on. Don't retry in a loop. The full list is in [references/tools.md](references/tools.md).

- `plan_required`: share `checkout_url` ($99/month Starter plan). Try again after the user starts it.
- `setting_up`: the brand is still setting up. Try again in a few minutes.
- Wrong brand, `workspace_required` or `workspace_not_found`: call `list_workspaces` with `query` set to the brand the user named, then call the tool again with that `id` as `workspace`.
- `limit_reached`: up to 3 jobs run at once per brand and 30 start per hour per person. Wait for one to finish, or stop an image ad, video ad or campaign job with `cancel_job`.

## Never

- Publish a page or site without an explicit ask.

## Examples

- "Make an image ad for our summer sale" → `create_image_ad` with that sentence as `brief`.
- "UGC video ad for the new serum, and let me review the script" → `list_video_formats`, then `create_video_ad` with the UGC `id` as `format` and `review: true`.
- "Landing page for my other brand, Lumi Coffee" → `list_workspaces` with `query: "Lumi Coffee"`, then `create_landing_page` with that `id` as `workspace`.
- "Run a Meta lead campaign at 20 a day" → `create_campaign` with that sentence as `brief`, `platform: "meta"`, `goal: "leads"` and `daily_budget: 20`.
- "How are my campaigns doing?" → `list_campaigns`.
- "Make the background darker" → `refine` on the last job.
- "Publish it" → `publish_site`, then share `site.live_url`.
