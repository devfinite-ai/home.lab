# Home Productivity architecture

## Component view

```mermaid
flowchart TB
    subgraph Hardware
        C[Controller buttons, pads, and encoders]
        L[Controller LEDs and display]
    end

    subgraph Device software
        D[Vendor driver and editor]
        P[MIDI mapper]
    end

    subgraph Home Productivity
        I[MIDI input monitor]
        R[Action router]
        G[Trigger and foreground guardrails]
        O[State and feedback coordinator]
    end

    subgraph Windows
        A[Core Audio and media controls]
        U[Window and input APIs]
        V[Legend overlay]
    end

    C --> D --> P
    P --> R
    D --> I --> R
    R --> G
    G --> A
    G --> U
    A --> O
    U --> O
    O --> V
    O --> L
```

## Trust boundaries

- MIDI input is untrusted event data. Values are validated before routing.
- Mappings select only named, allow-listed actions; they do not carry arbitrary
  scripts or shell commands.
- Input delivery requires an interactive, unlocked Windows session and a stable,
  verified foreground target.
- Audio, window, and feedback state are read back from Windows before the bridge
  reports success.
- Diagnostics must omit prompt text, credentials, private URLs, and local paths.

## Action lifecycle

```mermaid
sequenceDiagram
    participant H as Hardware
    participant B as Bridge
    participant G as Guardrails
    participant W as Windows
    participant F as Feedback

    H->>B: MIDI event
    B->>G: Parsed action and edge
    G->>G: Validate profile, debounce, target
    alt Accepted
        G->>W: Perform one allow-listed action
        W-->>B: Confirm resulting state
        B->>F: Update overlay or LED
        F-->>H: Visible hardware feedback
    else Rejected
        G-->>B: Suppression reason
    end
```

## Non-goals for v1

- Remote execution over the network
- Arbitrary command execution from mapping files
- Capturing application content or dictated prompts
- Bundling vendor drivers or proprietary controller templates
- Automating provider accounts or bypassing application permissions
