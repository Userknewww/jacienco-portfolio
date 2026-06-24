# Case Study — Glasses Messaging Bridge: Wearable Notifications via Beeper + Tailscale

## The challenge
I wanted my smart glasses to show messages from a unified inbox and let me reply, without exposing my desktop to the public internet. The constraint that shaped everything was that the glasses have no general network stack — they cannot be a normal API client. So the real problem was finding the correct integration boundary between a Bluetooth wearable, a phone app, and a desktop API, and then making that link private and reliable.

## Discovery
The first attempt failed silently, and the failure was instructive. The bridge app's API base URL was malformed — it carried a leftover localhost prefix and no port — so it could never have connected. Once that was corrected, the connection still failed: a direct test against the desktop API over the mesh address returned nothing, while localhost worked. That pinpointed the real issue — the API was bound only to the loopback interface and was invisible to the phone across the network. Neither problem was apparent from the app's error messages; both required testing the network path directly.

## The solution I designed
I established the integration as a three-hop path: glasses to phone over Bluetooth, phone to desktop API over a private Tailscale mesh network, and the desktop API into the unified inbox. I corrected the base URL to the desktop's real mesh address and the API's actual port. I enabled the desktop API's remote-access setting so it listened on all interfaces rather than localhost only, and fully relaunched it so the new binding took effect. I created an API token scoped to permit sensitive actions, which is what allows replies to be sent from the glasses rather than only read. The architecture diagram in the README shows the full three-hop path.

## The technical decisions
First, the phone as bridge: since the glasses cannot be the network client, the only correct design puts the phone app at the network boundary and the glasses on Bluetooth. Second, a private mesh over port-forwarding: routing phone-to-desktop traffic across Tailscale meant no router ports had to be opened and the link works from anywhere without exposing the desktop publicly. Third, binding the API beyond localhost: this was the decisive fix, and it is a classic forward-deployed lesson — a service that works locally can be completely unreachable across a network until it is explicitly told to listen there.

## The results
Messages from the unified inbox reach the glasses and short replies can be sent back, over a private network with no public exposure, reachable from anywhere on the mesh. The integration runs across three devices and two transport hops without opening the desktop to the internet.

## What I would do differently
I would have tested the raw network path — a direct request to the desktop API over the mesh address — before configuring the app, because that single test would have revealed both the malformed URL and the localhost-only binding immediately, instead of inferring them from an app that failed quietly. I would also add a conversation filter from the start, so the glasses only surface the threads that matter.
