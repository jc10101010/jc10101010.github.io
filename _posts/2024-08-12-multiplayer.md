---
layout:     page
title:      3D Multiplayer in Java using Custom Protocol, ViridianNetworking
summary:    A UDP game networking solution written in Java. Applied in a 3D multiplayer demo. 
categories: jekyll pixyll

---

Having played many 3D multiplayer games throughout my life and already programmed a 3D engine, I decided to create a **3D multiplayer demo**. In the process, I developed **Viridian Networking**, a Java-based networking library that, while optimized for 3D games, can be used in various contexts. This demo integrates with the previously created **TurquoiseGraphics** library.

![Multiplayer game image](/images/multiplayer-demo.png)

## Architecture

For this project, I chose the classic **Client-Server model**. The performance benefits of this approach are clear, and it also removes privacy concerns about revealing players' IP addresses. I opted for **UDP** (User Datagram Protocol) for data transmission, as it is significantly faster than TCP.

![Client Server Network](/images/client-server-network.png)

## Packet Definition

### Overall Protocol

The protocol was designed to be as human-readable as possible. I also ensured that all messages had a clear start and end, as UDP does not guarantee the delivery of packets in sequence. Here's the format I settled on:

```
START {CLIENT/SERVER} MESSAGE_CONTENT END
```

**Note:**  
- **CLIENT/SERVER** refers to the packet's sender.
- **MESSAGE_CONTENT** is the actual data being transmitted.

### A Typical Interaction

#### Interaction One: Joining a Game

##### Client Sends: The Join Packet
This is the first packet a client sends to the server, indicating its desire to join the game lobby. It’s like the client knocking on the server's door.

```
START CLIENT JOIN END
```

##### Server Sends: The You Joined Packet
This packet confirms that the client has successfully joined the lobby and can now request the current game state.

```
START SERVER YOUJOINED END
```

#### Interaction Two: Typical Data Transfer

##### Client Sends: The Set Position and Rotation Packet
The client sends this packet to inform the server of its current position and rotation in the game world.

```
START CLIENT SET POSITION 2 2 2 END
START CLIENT SET ROTATION 2.843 -0.342 0.8973 END
```

##### Client Sends: The Request Packet
This packet requests the current state of other players from the server.

```
START CLIENT REQUEST END
```

##### Server Sends: The State Position and Rotation Packet
In response, the server sends the position and rotation of other players. The first number indicates the player's name, which is essentially the ID assigned by the server.

```
START SERVER STATE POSITION 2 1 1 1.5 END
START SERVER STATE ROTATION 2 2.754 2.32 0.23 END
```

## Code Samples

### Server

The server in this project is **blocking**, meaning it waits continuously to receive a packet, then quickly responds before going back to waiting. Here's the main server code:

```java
/**
 * Performs the main loop of the server.
 * Waits to receive a packet and then responds.
 * Then it goes back to waiting.
 */
public void runServer() throws IOException {
    while (true) {
        // Block until a packet is received
        receivePacket();

        // Respond to the received packet
        generateResponsePacket();

        // Send response packets
        if (!responsePackets.isEmpty()) {
            for (Packet packet : responsePackets) {
                sendResponsePacket(packet);
            }
        }
    }
}
```

### Client

Unlike the server, the client is **non-blocking**, meaning it loops around 20 times per second. First, it sends any pending messages. If there are unprocessed packets from the server, it processes them. Here's the main client code:

```java
/**
 * Performs the main loop of the client.
 * Loops constantly, waiting to receive a packet,
 * and then processes it.
 */
public void run_client() throws IOException, InterruptedException {
    while (true) {
        // If already in the game
        if (has_joined_game) {
            // Send typical data transfer packets
            sendPacket(new ClientSetPositionPacket(player_position));
            sendPacket(new ClientSetRotationPacket(player_rotation));
            sendPacket(new ClientRequestPacket());
        } else {
            // Send a join packet
            sendPacket(new ClientJoinPacket());
        }

        // Receive any pending packets, NOT BLOCKING
        receivePackets();

        // Process any pending packets
        while (isPacketAvailable()) {
            processPacket(getFirstPacket());
        }
    }
}
```

### Packets

Since this library is written in Java, each packet is its own class and inherits from other packet classes. Here’s the code for the `ClientSetPositionPacket`:

```java
public class ClientSetPositionPacket extends ClientSetPacket {
    public Vertex position; // The position being sent in the packet

    public ClientSetPositionPacket(Vertex position) {
        clientSetType = clientSetTypeEnum.POSITION;
        this.position = position;
    }

    /**
     * Returns the packet as a string.
     * This method relies on the packet inheriting the wrapString
     * function from its parent class.
     */
    public String toString() {
        return wrapString(position.x + " " + position.y + " " + position.z);
    }
}
```

### Game Implementation

Here is an example of how packets are processed by the client in the specific implementation within the 3D demonstration:

```java
/**
 * Processes an individual Server State Position Packet,
 * moving the mentioned player's render object in-game.
 */
public void processPositionPacket(ServerStatePositionPacket packet) {
    // Get the data from the packet
    String name = packet.name;
    Vertex player_pos = packet.vertex;

    // If the other player already has an object in-game
    if (nameToObject.containsKey(name)) {
        // Find the object and move it
        RenderObject obj = nameToObject.get(name);  
        obj.setPosition(player_pos);
    } else {
        // Otherwise, create a new object for the player
        RenderObject obj = RenderObject.loadObject("data/monkey.obj", "player", 
            new InverseSqrShadow(new Color(0, 0, 255), game_scene), 
            new Vertex(0, 0, 0));

        // Add this object to our map and the scene
        nameToObject.put(name, obj);
        game_scene.addObject(obj);
    }
}
```

Find [the link to the GitHub here](https://github.com/jc10101010/ViridianNetworking).

### Future Plans:
- Improve edge-case error handling
- Develop a full lobby system
