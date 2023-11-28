---
title: Minecraft Protocol
description: A simple article on how to get started
date: 11.28.2023
---
> Note: I only talk about being a client, I might talk about being a server in the future

The Minecraft protocol is the way a Minecraft client and Minecraft server communicate over a TCP connection. Comparing it to other protocols I've worked with, this one isn't too complicated, but it is definitely  not the easiest one. Minecraft is a popular game and is commonly a starting point for coding/making games, so why not make coding a bot one too? 

## Prerequisite Knowledge
You will need to have a decent grasp on a programming language. I will show you my examples in [Deno/Node.js](https://deno.land), but if you are confident and know your stuff, and language will do (My own MC protocol implementation is in Rust!)

We will be working with the TCP network protocol. It's not complicated, but knowledge on the internet is helpful. 

The final thing you will need, and the most complicated, is knowledge on bitwise operations. This is the most important, because this is the backbone of operations on the data we send and receive. 

> Now, let's actually get started!

## Set up your environment
This varies from languages and build system, but here is how I do it with Deno:

**`deno.json`**
```json
{
  "tasks": {
    "dev": "deno run -A src/main.ts" // If you use Deno, you should probably be more restrictive on the permissions you give lol
  }
}
```

**`src/main.ts`**
```ts
console.log("Hello, world!");
```

> Also, a fair warning, I am very OOP oriented, so you will see me make a class for almost everything :)
## Build a basic connection manager
The way you manage a TCP connection varies from your languages standard library, but Deno has a fairly standard one built into it. I made this simple class to manage it all:
**`src/conn.ts`**
```ts
export interface ConnOptions {
  host: string,
  /** Defaults to 25565 */
  port?: number
}

/**
Represents a simple TCP connection manager for MC
*/
export class Conn {
  public conn?: Deno.TcpConn;
  public connected: boolean;

  private options: ConnOptions;

  /**
   * Creates a new connection manager
   * @param options The options for the connection
   */
  constructor(options: ConnOptions) {
    this.options = options;
    this.connected = false;
  }

  /**
   * Connects to the server
   */
  async connect() {
    this.connected = true;
    // needs the --allow-net permission
    this.conn = await Deno.connect({ hostname: this.options.host, port: this.options.port || 25565 });
  } 

  /**
   * Writes a packet to the connection
   * 
   * Note: This will not prepend the packet length
   * @param packet The packet to write
   */
  async write(packet: Uint8Array) {
    // Challenge: Can you rewrite this function to split the packet into multiple packets if it's too large?
    if (!this.connected) throw new Error("Not connected");
    await this.conn?.write(packet);
    // If your TCP library supports flushing, you should flush here (Deno has no flush method)
    // await this.conn?.flush(); 
  }

  /**
   * Reads a packet from the server
   */
  async read(): Promise<Uint8Array> {
    if (!this.connected) throw new Error("Not connected");
    const buf = new Uint8Array(1024);
    const n = await this.conn?.read(buf); // Read response length
    if (n == null) throw new Error("Connection closed");
    return buf.slice(0, n);
  }
}
```

## This article will be added to on a later date!
