<p align="center">
  <img src="https://david-dm.org/nurdism/battleye/status.svg" alt="Dependencies Status">
  <img src="https://david-dm.org/nurdism/battleye/dev-status.svg" alt="Dev Dependencies Status">
  <img src="https://david-dm.org/nurdism/battleye/peer-status.svg" alt="Peer Dependencies Status">
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/dm/battleye.svg" alt="Downloads"></a>
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/v/battleye.svg" alt="Version"></a>
  <a href="https://www.npmjs.com/package/battleye"><img src="https://img.shields.io/npm/l/battleye.svg" alt="License"></a>
  <a href="https://discord.gg/Kzkd6V3" ><img src="https://discordapp.com/api/guilds/428366869993488401/widget.png" alt="Chat on discord"><a/>
</p>

# battleye

> Battleye rcon client built in nodejs.

## Example usage:
```js
import * as readline from 'readline'
import { readCfg, Socket } from 'battleye'

readCfg(process.cwd())
  .then(cfg => {
    console.log(cfg)

    const socket = new Socket()
    const connection = socket.connection({
      password: cfg.rconpassword,
      ip: cfg.rconip,
      port: parseInt(cfg.rconport, 10)
    })

    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    })

    socket.on('listening', (socket) => {
      const addr = socket.address()
      console.log(`Socket listening on ${typeof addr === 'string' ? addr : `${addr.address}:${addr.port}`}`)
    })

    socket.on('received', (resolved, packet, buffer, connection, info) => {
      console.log(`received: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    socket.on('sent', (packet, buffer, bytes, connection) => {
      console.log(`sent: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    socket.on('error', (err) => { console.error(`SOCKET ERROR:`, err) })

    connection.on('message', (message, packet) => {
      console.log(`message: ${connection.ip}:${connection.port} => message: ${message}`)
    })

    connection.on('command', (data, resolved, packet) => {
      console.log(`command: ${connection.ip}:${connection.port} => packet:`, packet)
    })

    connection.on('disconnected', (reason) => {
      console.warn(`disconnected from ${connection.ip}:${connection.port},`, reason)
    })

    connection.on('connected', () => {
      console.error(`connected to ${connection.ip}:${connection.port}`)
    })

    connection.on('message', (message, packet) => {
      console.error(`message: ${connection.ip}:${connection.port} => ${message}`)
    })

    connection.on('debug', console.log)

    connection.on('error', (err) => {
      console.error(`CONNECTION ERROR:`, err)
    })

    rl.on('line', input => {
      connection
        .command(input)
        .then(response => {
          console.log(`response: ${connection.ip}:${connection.port} => ${response.command}\n${response.data}`)
        })
        .catch(console.error)

      console.log(`send: ${connection.ip}:${connection.port} => ${input}`)
    })
  })
  .catch(err => { console.error(`Error reading config:`, err) })
```