---
title: "Using Socket.IO on Adonis v6"
datePublished: Wed Feb 21 2024 04:29:47 GMT+0000 (Coordinated Universal Time)
cuid: clsvapgry000009l63lmff0di
slug: socketio-adonis-v6
tags: adonisjs

---

```typescript
// app/services/socket_service.ts

import { Server } from 'socket.io'
import server from '@adonisjs/core/services/server'

class SocketService {
  io?: Server

  #booted: boolean = false

  boot() {
    if (this.#booted) {
      return
    }

    this.io = new Server(server.getNodeServer(), {
      cors: {
        origin: '*',
      },
    })

    this.#booted = true
  }
}

export default new SocketService()
```

```typescript
// start/socket.ts

import SocketService from '#services/socket_service'

SocketService.boot()

SocketService.io?.on('connection', (socket) => {
  socket.emit('connected', { connect: true })
})

export const io = SocketService.io
```

```typescript
// providers/app_provider.ts

import type { ApplicationService } from '@adonisjs/core/types'

export default class AppProvider {
  constructor(protected app: ApplicationService) {}

  register() {}

  async boot() {}

  async start() {}

  async ready() {
    if (this.app.getEnvironment() === 'web') {
      await import('#start/socket')
    }
  }

  async shutdown() {}
}
```