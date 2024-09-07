---
layout:     page
title:      3D Multiplayer in Java, ViridianNetworking
summary:    A multiplayer demo using the 3D engine that I built previously
categories: jekyll pixyll
---

Having played many 3D multiplayer games in my life, and having programmed a 3D engine already.
I decided to undertake the process of making a __3D multiplayer demo__. While doing this I created __VeridianNetworking__
a networking library for Java, that is not tied to its 3D roots, although it is definitely most useful
for that purpose.

![Multiplayer game image](/images/multiplayer-demo.png)


## Architecture

For this project I decided to go with the classic __Client-Server model__. 
The performance benefits of this approach are self evident, and it also removes any 
privacy concerns about knowing other player's ip addresses. I also decided to use UDP 
for the data transmission as it is much more swift than TCP.

![Client Server Network](/images/client-server-network.png)

## Packet Definition
Defining the packets for this project was challenging, but here are the final packets I decided on:

### Overall protocol
For this project I aimed to make the protocol as human readable as possible. I also had to ensure that
all messages that were sent had a clear start and end as UDP does not guarantee that packets
will be transmitted from start to end. Here is the format I decided on:
{% highlight  lineanchors %}
START {CLIENT/SERVER} MESSAGE_CONTENT END

Note: 
CLIENT/SERVER refers to the sender of the packet
MESSAGE_CONTENT refers to the actual data being sent
{% endhighlight %}

### A typical interaction
####      Interaction One: Joining A Game
##### Client Sends: The Join Packet
This is the first packet a client sends to the server, to indicate that it wants 
to be part of the lobby. Essentially this packet is like the computer knocking on the door.
{% highlight  lineanchors %}
START CLIENT JOIN END
{% endhighlight %}
##### Server Sends: The You Joined Packet
This is the packet the server sends to the client to indicate that
it is now part of the lobby and can request the current game state data.
{% highlight  lineanchors %}
START SERVER YOUJOINED END
{% endhighlight %}




####      Interaction Two: Typical Data Transfer
##### Client Sends: The Set Position and Rotation Packet
This is the packet the client sends to the server to tell the server its current
position and rotation in the world. 
{% highlight  lineanchors %}
START CLIENT SET POSITION 2 2 2 END
START CLIENT SET ROTATION 2.843 -0.342 0.8973 END
{% endhighlight %}
##### Client Sends: The Request Packet
This is the packet the client sends to the server to ask for the current state of the other players
in the game.
{% highlight  lineanchors %}
START CLIENT REQUEST END
{% endhighlight %}
  
##### Server Sends: The State Position and Rotation Packet
This is the packet the server sends to the client to inform the client what the position and rotation of some player is.
The server sends one of these packets for each player other than the client-player. The first number indicates
the "name" of the player, which is just the id assigned by the server.
{% highlight  lineanchors %}
START SERVER STATE POSITION 2 1 1 1.5 END
START SERVER STATE ROTATION 2 2.754 2.32 0.23 END
{% endhighlight %}

## Code Samples 
### Server
The server in this project is __blocking__, constantly waiting to receive a 
packet so it can quickly respond with some packets of its own. 
Here is the main server code:
{% highlight java lineanchors %}

/**
* Performs the main loop of the server 
* Waits to receieve a packet and then responds.
* then it goes back to waiting.
*/
public void runServer() throws IOException {
    while (true) {
        //Block until a packet is receieved
        receivePacket();

        //Respond to packet receieved
        generateResponsePacket();

        //Send response packets
        if (responsePackets.size() != 0) {
            for (Packet packet : responsePackets) {
                sendResponsePacket(packet);
            }
        }
    }
}

{% endhighlight %}

### Client

Unlike the server the client is __not blocking__, this means
that the client is looping ~20 times a second. First the 
client sends any pending messages. Then if it has 
new packets from the server that have not been processed it will process them.
Here is the main client code:

{% highlight java lineanchors %}

/**
* Performs the main loop of the client 
* loops constantly waiting to receieve a packet 
* and then processes it.
*/
public void run_client() throws IOException, InterruptedException{
    while (true) {
        //If already in game
        if (has_joined_game) {
            //Send typical data transfer packets
            sendPacket(new ClientSetPositionPacket(player_position));
            sendPacket(new ClientSetRotationPacket(player_rotation));
            sendPacket(new ClientRequestPacket());
        } else {
            //Send a join packet
            sendPacket(new ClientJoinPacket());
        }
        
        //Receieve any pending packets, NOT BLOCKING
        receivePackets();

        //Process any packets that were pending
        while (isPacketAvailable()) {
            processPacket(getFirstPacket());
        }

    }
}

{% endhighlight %}

### Packets
As this library was written in Java, all packets are their own Class and inherit from other packets. 
Heres the code for defining the ClientSetPositionPacket packet: 
{% highlight java lineanchors %}

public class ClientSetPositionPacket extends ClientSetPacket {
    public Vertex position; //The position being sent in the packet

    public ClientSetPositionPacket(Vertex position) {
        clientSetType = clientSetTypeEnum.POSITION;
        this.position = position;
    }

    /**
    * This function returns the packet if it were a string.
    * It relies on the packet inheriting the wrapString function from
    * it's parents.
    */
    public String toString() {
        return wrapString(position.x + " " + position.y + " " + position.z);
    }
}

{% endhighlight %}

### Game Implementation
Here is an example of how the packets are processed by the client
 in the specific implementation within the 3D demonstration:

{% highlight java lineanchors %}

/**
* Processes an individual Server State Position Packet, moving the mentioned player's render object in game
*/
public void processPositionPacket(ServerStatePositionPacket packet) {
    //Get the data from the packet
    String name = packet.name;
    Vertex player_pos = packet.vertex;

    //If other player already an object in game
    if (nameToObject.keySet().contains(name)) {

        //Find the object of the other player, and move it
        RenderObject obj = nameToObject.get(name);  
        obj.setPosition(player_pos);

    } else {

        //Otherwise make a new object that represents the player
        RenderObject obj = RenderObject.loadObject("data/monkey.obj", "player", new InverseSqrShadow(new Color(0,0,255), game_scene), new Vertex(0,0f,0));

        //Add this object to our map and to the Scene
        nameToObject.put(name, obj);
        game_scene.addObject(obj);

    }
}

{% endhighlight %}

Find [the link to the github here.](https://github.com/jc10101010/ViridianNetworking)

### Future Plans: 
  * Better edge case error handling
  * A full lobby system
