Peer to peer starter
====================

A toy to explore simple hub and spoke peer to peer applications built on WebRTC between supporting browsers.

Features
--------

- Single "host" hosting a "game"
- Multiple "players" can join the game
- All web based, only browsers needed
- All peer-to-peer client based state only, no server state (web app served as static files)
- WebRTC connection negotiated by a (shared) app agnostic signalling server

Communication
-------------

1. Load page (/index.html)
2. By default start "Host", give link to "Player" URL (/player.html)
3. For host: create and display game, provide link to Player to join Game (/player.html?game=ID)
3. For player: enter game ID to join (if not provided in URL)
4. Player starts sending "commands" to host
5. Host continually processes commands, updating and displaying game state

```mermaid
sequenceDiagram
    participant H as Host
    participant HP as Player peer connection
    participant S as Signalling room
    participant PH as Host peer connection
    participant P as Player

    activate H
    H-->>H: Create game and update link

    H->>S: (Join)
    activate S

    Note over P: Player opens (QR or URL)
    activate P

    P->>S: (Join)
    note over P,H: Both sides have joined signalling room so can communicate

    P-->>PH: Create
    activate PH

    P->>S: ping
    S->>H: ping

    H-->>HP: Create

    H->>S: pong
    S->>P: pong

    note over P, H: Both sides know the other's listening, so we negotiate WebRTC session description
    P->>PH: createOffer()
    PH->>P: <offer>
    P->>S: <offer>
    S->>H: <offer>
    H->>HP: setRemoteDesc(<offer>)
    H->>HP: createAnswer()
    HP->>H: <answer>
    H->>HP: setLocalDesc(<answer>)
    H->>S: <answer>
    S->>P: <answer>
    P->>PH: setRemoteDesc(<answer>)

    note over P,H: Session params shared, so now exchange ICE candidates until WebRTC connects
    HP->>H: <host candidate>
    H->>S: <host candidate>
    S->>P: <host candidate>
    P->>PH: <host candidate>
    note over P,H: (or other way)
    PH->>P: <player candidate>
    P->>S: <player candidate>
    S->>H: <player candidate>
    H->>HP: <player candidate>

    note over H,P: Once peers are connected, no need for signalling
    deactivate S
    H-->HP: <host event>
    HP->>PH: <host event>
    P-->PH: <player event>
    PH->>HP: <player event>
```

Tech stack
----------

- SPA
- Vanilla JS where possible
    - Reset UI each state change?
    - WebRTC for Host-Player connection
    - WebSocket signalling for WebRTC negotiation
    - Browser motion API for events
    - Single `main.js` script loaded as module in HTML
    - Other `.js` files imported normally, browser will load
- `qr-code` library (single WebComponent)

Development
-----------

**Dependencies**

- Make - for some utility tasks
- NodeJS - to serve files locally
- openssl - to generate self signed certs

**HTTPS**

There is no build step but in order to use the browser motion API there's a need to access the client app over HTTPS.  Easiest way to do that is make sure your dev machine announces itself via mDNS (MacOS has Bonjour, Linux has Avahi) then use "<hostname>.local":

```bash
DUMMY_HOSTNAME=<changethis>.local make server
```
