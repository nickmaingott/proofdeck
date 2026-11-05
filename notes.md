# Real-Time Multiplayer Networking: what I learned building Shard Scramble

These are my notes from building Shard Scramble, a browser arena game where 2–8 players join a room with a 4-letter code.

## 1. How the game is organized

**Host-authoritative model** — one player's browser (the host) runs the only true copy of the game, and every other player just sends inputs and draws what the host reports.
I chose this so players cannot disagree about who picked up a shard: the host decides every collision and every score.
The downside is that the host is a single point of failure, so if the host closes the tab the round ends for everyone.

**Fixed timestep** — the simulation advances in equal steps (1/60 of a second) no matter how fast the screen redraws.
A fixed timestep keeps physics the same on a fast laptop and a slow phone.

**Tick rate** — how many times per second the host sends a snapshot of the game state to players.
Shard Scramble simulates at 60 Hz but sends snapshots at 20 Hz to save bandwidth, and only 1 snapshot per second in the lobby.

**Snapshot** — a compact message with every player's position, score and status plus the list of shards on the board.
Positions are rounded to whole numbers to keep each snapshot small.

## 2. Getting messages between devices

**WebRTC** — a browser technology that lets two browsers send data directly to each other (peer-to-peer) after a short setup through a signaling server.
My first version used WebRTC through the PeerJS library.

**NAT (Network Address Translation)** — the way a home or school router lets many devices share one public IP address, which also hides each device from the outside internet.
NAT is the main reason peer-to-peer connections are hard: neither browser knows the other's real address.

**STUN server** — a public server that tells a browser what its public IP address and port look like from the outside, so a peer can try to reach it.
On my network, only "host" ICE candidates appeared and no STUN result came back, which showed that UDP traffic to STUN was blocked.

**ICE candidate** — one possible network address (local, public via STUN, or relayed via TURN) that WebRTC tries when connecting two peers.

**TURN server** — a relay server WebRTC falls back to when a direct connection is impossible; it works almost everywhere but someone has to pay to run it.

**WebSocket** — a long-lived, two-way connection between a browser and a server that runs over the same ports as normal HTTPS traffic.
Because WebSockets look like regular web traffic, they get through school and office firewalls that block WebRTC.

**MQTT** — a lightweight publish/subscribe messaging protocol where clients publish messages to named topics and a broker forwards them to every subscriber.
I switched Shard Scramble to MQTT over secure WebSockets using a free public broker, which fixed the connection problem without needing my own server.

**Broker** — the MQTT server in the middle that receives every published message and forwards it to the clients subscribed to that topic.

**Topic** — the address an MQTT message is published to; each room uses its own topics such as `room/up` for player inputs and `room/down` for host snapshots.

**Retained message** — an MQTT message the broker stores and hands to anyone who subscribes later.
The host publishes a retained "meta" message so a joining player instantly learns that the room exists.

**Last will message** — a message the broker publishes automatically if a client disconnects without saying goodbye.
The host's last will clears the room's meta message, so dead rooms disappear on their own.

**QoS (Quality of Service)** — the MQTT delivery guarantee level: QoS 0 is "at most once" and QoS 1 is "at least once".
I send frequent snapshots with QoS 0 because a lost one is replaced 50 ms later, but important events like "round over" use QoS 1.

## 3. Hiding lag

**Latency** — the time a message takes to travel from one device to another.
From my network the round trip through the broker was about 0.5 seconds, which made movement feel sluggish at first.

**Client-side prediction** — the player's own device moves their character immediately when a key is pressed instead of waiting for the host to confirm.
Prediction makes controls feel instant even with half a second of latency.

**Reconciliation** — correcting the predicted position when the host's authoritative snapshot arrives, by starting from the host's position and replaying the inputs the host has not processed yet.
Each input carries a sequence number, and the host reports the last sequence number it applied (the "ack").

**Interpolation** — drawing other players smoothly between the last two snapshots instead of letting them jump 20 times per second.

## 4. Lessons about building with AI

Testing on two separate browsers found bugs that a single tab never shows.
A background browser tab slows its timers down, so my first timeout of 8 seconds kicked players who simply switched apps; I raised it to 20 seconds and added automatic rejoin between rounds.
Describing a bug precisely ("only host ICE candidates appear, no srflx") led to the right fix much faster than saying "it doesn't connect".
