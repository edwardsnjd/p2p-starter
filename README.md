Peer to peer starter
====================

A toy to explore simple hub and spoke peer to peer applications built on WebRTC between supporting browsers.

Features
--------

- Web app served as static files only i.e. no server state/database etc.
- One peer acts as "host" hosting a "game" (this is not required but fits the hub and spoke P2P game use case)
- Multiple "players" can join the "game", and communicate P2P with the host
- All web based, only browsers needed
- All peer-to-peer client based state only, no server state
- WebRTC connection negotiated via temporary WebSocket connection to a signalling server

Notes
-----

- Signalling is required to set up WebRTC
- Signalling server is application agnostic, just broadcasts to all peers subscribed to the "game" room
- The signalling could be via QR codes and scanning if all peers support it
- To allow for dumb hosts (e.g. TVs) this starter uses a WebSocket signalling approach

Tech stack
----------

- Vanilla JS modules
- No build step
- WebSocket signalling for WebRTC negotiation
- WebRTC for Host-Player connection
- Browser motion API for events
- (for optional QR) `qr-code` library (single WebComponent) loaded via CDN

Communication
-------------

Two web browsers end up communicating P2P after the following flow:

1. [Host] Load page (/host.html)
2. [Host] Create and display game, provide link to Player to join Game (/player.html?game=ID)
3. [Player] Scan QR code navigate to provided URL
4. [Both] Negotiate a P2P connection using temporary connection to signalling
5. [Both] Drop the signalling connection and communicate purely P2P

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
    PH->>P: offer
    P->>S: offer
    S->>H: offer
    H->>HP: setRemoteDesc(offer)
    H->>HP: createAnswer()
    HP->>H: answer
    H->>HP: setLocalDesc(answer)
    H->>S: answer
    S->>P: answer
    P->>PH: setRemoteDesc(answer)

    note over P,H: Session params shared, so now exchange ICE candidates until WebRTC connects
    HP->>H: host candidate
    H->>S: host candidate
    S->>P: host candidate
    P->>PH: host candidate
    note over P,H: (or other way)
    PH->>P: player candidate
    P->>S: player candidate
    S->>H: player candidate
    H->>HP: player candidate

    note over H,P: Once peers are connected, no need for signalling
    deactivate S
    H-->HP: host event
    HP->>PH: host event
    P-->PH: player event
    PH->>HP: player event
```

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
