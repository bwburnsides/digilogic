## Simulation Protocol

The goal is a message protocol between a simulation backend and UI front-end that would allow for performance-limited devices (such as mobile devices) to use a resource-rich networked machine for increased performance.

- UDP as transport to avoid latency associated with retries in TCP
- Latency associated with Nagle's algorithm *could* be avoided with `SO_NODELAY`.
- Control packets to implement ack / retry on top of UDP to ensure that state data isn't dropped
    - This is important to ensure wires aren't improperly driven / ROMs improperly written
- Visuals can be updated at a lower frequency than the actual simulation state - for example wire states could be drawn with a duty cycle representation.
- Could instead have two channels:
    - UDP update channel: wire snapshots
    - TCP command channel: everything else

### Packets
```rs
enum UiUpdateKind {
    DutyCycle,
    RealTime(u16), // Max update frequency
    SetFrequency(u16),
}

enum SimulationControlCommand {
    FreeRun(u16),  // Frequency
    Step,
    Pause,
    UpdateUiStyle(UiUpdateKind),
}

// Sent from UI to Sim
struct SimulationControl {
    command: SimulationControlCommand
}
```

This message could be used for actual internal simulation state updates or just visual updates.
```rs
type ElementId = u64;

enum ElementState {
    TriState,
    High,
    Low,
    Error,
}

// Sent from Sim to UI
struct SimulationUpdate {
    elements: [(ElementId, ElementState); 1024]
}
```

Used to update the values stored in simulated memory devices. The above message should use some type of enumerate for state-kinds with the below being a variant. That way wire updates and memory updates could be done in one message-type.
```rs
type MemoryElementAddress = u32;
type MemoryElementData = u64;

struct MemoryElementUpdate {
    items: u8
    addresses: [MemoryElementAddress; u8::MAX],
    values: [MemoryElementData; u8::MAX]
}
```
