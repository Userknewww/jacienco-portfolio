# Case Study — Hermes: A Self-Hosted Multi-Agent Messaging Gateway

## The challenge
Most AI agents are demos. They run in a terminal window, answer a few prompts, and stop existing the moment the window closes. I wanted the opposite: an agent that lives inside the tools I already use every day — Slack and Telegram — and stays available around the clock without me restarting it. That meant moving from "a script I run" to "a service that runs itself," which is a meaningfully harder problem involving a Linux host, process supervision, platform integrations, and cost control.

## Discovery
The first working version was fragile in three specific ways I did not anticipate. The model provider I started with rejected every request with a "model not supported" error, which forced a provider migration mid-build. The agent then ran out of usage credits silently, because external agent calls billed against a different balance than I assumed. And the process only ran while I was connected to the server. Each of these was invisible until the system was actually expected to stay up on its own — which is exactly the kind of gap that only shows up in a live environment, not a demo.

## The solution I designed
I deployed the gateway as a supervised background service on a Linux VPS, registered with the operating system so it starts on boot and restarts automatically on failure. I migrated the model provider to Anthropic with OAuth authentication and selected a mid-tier model for sustainable 24/7 operation. I created a single private Slack channel as the agent's permanent home and connected Telegram as a second front end, so the same agent is reachable from a desktop channel and a phone. The architecture diagram in the project README shows the message path from each client, through the gateway, to the model and back.

## The technical decisions
First, service over script: registering the gateway with the host's init system was the decision that turned a demo into infrastructure, because it removed me as the thing keeping it alive. Second, model and spend cap: I chose a mid-tier model and set a hard monthly spend ceiling rather than a larger model with open-ended cost, because the binding constraint on an always-on agent is cost per call, not peak quality. Third, a single home channel: consolidating the agent into one channel gave me one predictable control surface and a clean foundation for adding more agents later without losing track of which bot does what.

## The results
The gateway runs unattended and has survived server reboots without manual intervention, answering from both Slack and Telegram. Spend is bounded by a fixed monthly cap, so the system cannot produce a runaway bill. Most importantly, the operating model changed: the agent is now something that stays up on its own rather than something I have to start. (Fill in your own measured uptime window here before publishing — for example, the number of days it has run since the last manual restart.)

## What I would do differently
I would have set up process supervision and the spend cap on day one, before wiring up the platforms, instead of discovering the need for both under pressure. I would also have added structured logging and a simple health check from the start, so that diagnosing a failure did not depend on reading raw service logs after the fact. Both are first-day decisions in hindsight, not week-two retrofits.
