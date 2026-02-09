---
trigger: always_on
---

# Common Patterns

## API Response Format

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

## Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## Skeleton Projects

When implementing new functionality:
1. Search for battle-tested skeleton projects
2. Use parallel agents to evaluate options:
   - Security assessment
   - Extensibility analysis
   - Relevance scoring
   - Implementation planning
3. Clone best match as foundation
4. Iterate within proven structure

## Local First Pattern

### Sync Types

```typescript
type SyncStatus = 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED'

interface LocalEntity {
  _localId: string
  _status: SyncStatus
  _lastUpdated: number
}

interface SyncQueueItem<T> {
  id: string
  action: 'CREATE' | 'UPDATE' | 'DELETE'
  payload: T
  timestamp: number
  retryCount: number
}

```

### Local First Repository Implementation

```typescript
abstract class LocalFirstRepository<T extends LocalEntity> implements Repository<T> {
  constructor(
    private localStore: SQLiteDatabase,
    private remoteClient: Repository<T>,
    private syncQueue: QueueManager
  ) {}

  async findAll(filters?: Filters): Promise<T[]> {
    const localData = await this.localStore.find(filters)
    
    if (navigator.onLine) {
      this.syncBackground(filters)
    }

    return localData
  }

  async create(data: CreateDto): Promise<T> {
    const localId = crypto.randomUUID()
    
    const optimisticData = {
      ...data,
      _localId: localId,
      _status: 'PENDING',
      _lastUpdated: Date.now()
    } as T

    await this.localStore.insert(optimisticData)

    await this.syncQueue.enqueue({
      id: localId,
      action: 'CREATE',
      payload: data,
      timestamp: Date.now(),
      retryCount: 0
    })

    this.triggerSync()

    return optimisticData
  }

  async update(id: string, data: UpdateDto): Promise<T> {
    await this.localStore.update(id, {
      ...data,
      _status: 'PENDING',
      _lastUpdated: Date.now()
    })

    await this.syncQueue.enqueue({
      id,
      action: 'UPDATE',
      payload: data,
      timestamp: Date.now(),
      retryCount: 0
    })

    this.triggerSync()

    return this.localStore.findById(id)
  }

  private async triggerSync(): Promise<void> {
    if (!navigator.onLine) return

    const pendingItems = await this.syncQueue.getPending()

    for (const item of pendingItems) {
      try {
        await this.syncQueue.updateStatus(item.id, 'SYNCING')
        
        let result
        switch (item.action) {
          case 'CREATE':
            result = await this.remoteClient.create(item.payload)
            await this.localStore.update(item.id, {
              ...result,
              _status: 'SYNCED'
            })
            break
          case 'UPDATE':
            result = await this.remoteClient.update(item.id, item.payload)
            await this.localStore.update(item.id, {
              ...result,
              _status: 'SYNCED'
            })
            break
        }

        await this.syncQueue.remove(item.id)
      } catch (error) {
        await this.syncQueue.updateStatus(item.id, 'FAILED')
      }
    }
  }

  private async syncBackground(filters?: Filters): Promise<void> {
    try {
      const remoteData = await this.remoteClient.findAll(filters)
      await this.localStore.bulkUpsert(remoteData.map(item => ({
        ...item,
        _status: 'SYNCED',
        _lastUpdated: Date.now()
      })))
    } catch (error) {
      console.error(error)
    }
  }
}

```