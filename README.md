# Meridian Framework

Meridian is a framework for infrastructure identity and deployment topology. It gives application teams and operators a shared vocabulary for naming environments, placing deployments, and assigning state and dependencies to their owners.

<!-- meridian-quick-reference:start -->

## Identity dimensions

Use the relative order **Service, Unit, World, Realm, Space, Area, Cell** when presenting these dimensions. An operation needs only the dimensions it consumes. Document the fixed enclosing scope whenever an identifier omits a dimension.

| Dimension | Meaning |
| --- | --- |
| **Service** | A runtime service of a Unit, such as an API server or worker. One Unit can run several Services, and each Service can run several processes or replicas. |
| **Unit** | A component such as an application backend, frontend, or infrastructure unit. |
| **World** | The product or system. |
| **Realm** | An operational purpose within a World, such as production or collaborative development. |
| **Space** | A logical application and state namespace within a World and Realm. |
| **Area** | An infrastructure placement scope within a World and Realm. |
| **Cell** | An installation or logical deployment target that can occupy one or more Areas. Its slug is opaque. |

An **availability zone (AZ)** is a physical failure domain. A Cell may occupy one AZ or span several AZs or datacenters. Record provider zone names and shared physical risks separately from logical placement.

## Qualified identities

| Identity | Qualified tuple | URI pattern |
| --- | --- | --- |
| Logical application environment | World + Realm + Space | `{world}-{realm}-{space}` |
| Server | World + Realm + Area + Node ID | `{realm}-{area}-{nodeId}.{world}` |
| Deployment Cell | World + Realm + Cell | `{world}-{realm}-{cell}` |

Record each Cell's Areas in placement configuration. The same Cell URI identifies the target across all its Areas. In the server URI, `nodeId` identifies the node within its World, Realm, and Area.

A logical deployment associates one Space with one Cell in the same World and Realm. Its qualified tuple is World + Realm + Space + Cell. A Space may deploy to several Cells, and a Cell may host several Spaces. Each environment has at most one logical deployment per Cell. Its Units, Services, nodes, replicas, and successive releases belong to that deployment.

### State ownership and Cell roles

Application state belongs to its World, Realm, and Space. Scope placement resources to their qualified target. A dependency that can safely isolate Spaces can serve the whole World and Realm as one logical instance.

The Cell slug is unique within its World and Realm and may repeat in another World or Realm. Cell slugs can resemble provider or zone names. Resolve roles and physical placement from the target configuration. Master and replica responsibility, membership, and the descriptive role `cluster` are installation properties. Changing these properties preserves Cell identity. Each controller defines its supported roles and operations.

<!-- meridian-quick-reference:end -->

```mermaid
flowchart LR
    WR["World + Realm"] --> E["Application environment<br/>World + Realm + Space"]
    WR --> C["Deployment Cell<br/>World + Realm + Cell"]
    E -->|zero or more| D["Logical deployment<br/>World + Realm + Space + Cell"]
    C -->|zero or more| D
    D --> U["Units and Services"]
    C -. logical placement .-> A["One or more Areas"]
    C -. physical placement .-> Z["Availability zones"]
```

## Adopting Meridian

See [Kerbmark’s adoption path](https://github.com/d1ys3nk0/kerbmark#use-the-frameworks-together) when combining Meridian with the workspace, engineering, and Ansible frameworks.

For offline use, copy the marked quick-reference block verbatim into one Markdown file in the consumer repository. Record the full Meridian commit and link back to this README. Route basic questions to the local copy and use this README for detailed guidance. When refreshing a copy, extract the block from the chosen commit and compare the copied text with that revision. Document project-specific slugs, inputs, identifier syntax, topology, and exceptions with their owners. The identity tuples describe scope. Each project defines its own configuration contract and each target system defines its separators, escaping, and length limits.

Use the recommendations below when introducing or explicitly migrating an identity. Before renaming an existing slug, account for consumers, named resources, and recovery procedures.

## Recommended slugs and readable names

Use the same lowercase slug for the same meaning across projects. Uppercase display abbreviations such as EDG and HST and the full names below explain the slugs. Extend the registry locally for product identities, tenants, sites, and exceptions. Projects define which values their parsers accept.

When choosing a World slug, prefer a short, distinctive abbreviation of the product name, such as `uncrn` for Unicorn. Document the full name alongside the slug and reuse the slug consistently so identifiers stay short and easy to search for.

### Realm and Space slugs

| Realm (`OPS_REALM`) | Full name | Intended meaning | Recommended Spaces (`OPS_SPACE`) |
| --- | --- | --- | --- |
| `prd` | Production | Production operation. | `hq` (Headquarters) for the main production application environment. |
| `stg` | Staging | Pre-production acceptance with production-like configuration and, where explicitly permitted, real data. | — |
| `dev` | Development | Collaborative development and testing, normally delivered to a centrally reachable, shared environment. | `qa01`, `qa02`, … (Quality assurance 01, 02, …) for independently identified collaborative QA environments. |
| `lab` | Laboratory | Individual experimentation and development, normally local and private to one person. | `sbx` (Sandbox) for interactive development and experimentation. `tst` (Test) for dedicated automated-test state and fixtures. |

### Slugs shared across realms

The input names below show how existing `OPS_*` contracts can map to Meridian’s dimensions.

| Existing identity input | Recommended slug | Full name | Intended meaning |
| --- | --- | --- | --- |
| `OPS_UNIT` | `core` | Core | Main application backend component. |
| `OPS_UNIT` | `web` | Web | Web frontend component. |
| `OPS_UNIT` | `ops` | Operations | Operational tooling component, where independently identified. |
| `OPS_WORLD` | `<product>` | Registered product or system name | Stable project-owned product slug, such as `uncrn` for Unicorn. |
| `OPS_AREA` | `edg` | Edge | User-operated laptops, desktops, and workstations. |
| `OPS_CELL` | `hst` | Host | Installation whose application processes execute directly in the host operating system. |
| `OPS_CELL` | `vgt` | Vagrant | Installation in one or more virtual machines managed through Vagrant. |
| `OPS_CELL` | `dkp` | Dokploy | Dokploy-managed installation with project-defined orchestration and placement. |
| `OPS_CELL` | `k8s` | Kubernetes | Kubernetes-based installation, with the actual cluster/namespace boundary explicitly defined by the project. |

Existing `OPS_SERVICE` contracts can use the following Service vocabulary.

| Recommended slug | Full name | Intended meaning |
| --- | --- | --- |
| `api` | Application programming interface | Runtime API service of a Unit. |
| `web` | Web | Runtime web-serving service of a Unit. |
| `worker` | Worker | Background processing service of a Unit. |
| `server` | Server | Generic server service where a more specific service identity is unnecessary or the project already uses this value. |

Unit `web` and Service `web` belong to different dimensions. Service names, availability-zone identifiers, release and execution metadata, image references, and addresses follow their owning contracts. Keep provider-defined AZ identifiers intact rather than replacing them with Cell slugs.

### Development and laboratory purposes

Use `dev` for collaborative development, such as centrally hosted QA Spaces. Use `lab` for individual work, typically with native processes, Compose, or local virtual machines. Starting an application locally creates a deployment association for that environment and target.

For local individual development across projects, use Realm `lab`, Area `edg`, Space `sbx` for interactive sandbox work and Space `tst` for automated tests when introducing or explicitly migrating those identities. Use a distinct Space for a distinct state namespace.

Choose the Realm by operational purpose, independently of the hosting technology. A private remote laboratory can use `lab`. A workstation can host a shared `dev` environment with explicit access and ownership boundaries. Use `stg` for pre-production acceptance.

### Placement mnemonics and uniqueness

Resolve the actual topology from configuration. The recommended Cell slugs give readers a naming mnemonic. Choose `hst` for a native application installation, including when supporting dependencies run in Compose. Choose `cmp` for a Compose-based application installation. VM nodes, container replicas, and Kubernetes nodes can all be members of one Cell.

A bare `hst`, `cmp` or `vgt` is appropriate when unique within World + Realm. Two independently identified workstations in `(uncrn, lab)` need distinct stable Cell slugs, such as `hst-ws01` and `hst-ws02`, even if they occupy different Areas. A Cell can retain its identity as its implementation changes.

## State, placement and metadata

Scope application-owned databases, buckets, queues, and other application namespaces by World + Realm + Space, adding Unit or module distinctions where needed. The state identity persists across deployments of the same Space. Replication, independent physical copies and shared access require explicit data ownership and routing contracts.

Scope placement-owned resources to their owner. Use the server identity for host management and the qualified Cell identity for resources shared by the installation. Resources owned by a particular application deployment may need both Space and the qualified Cell target. Project-specific shortened context, lock or history keys must document their fixed scope and collision limits.

Moving data, switching endpoints, promoting a database, and changing public ingress each require an explicit operation and verification.

Release versions, build identifiers, image references, deployment execution tags and launch identifiers describe artifacts or executions of a logical deployment. A new launch or release preserves the deployment association even when its Services or replica counts change.

### Shared dependencies and Space isolation

Choose the ownership scope by who controls the dependency’s lifecycle and state, including configuration, startup, shutdown, reset, and recovery.

| Scope | Lifecycle and state ownership |
| --- | --- |
| Realm-shared | The World and Realm own the logical dependency. Each Space owns the application state inside its enforced boundary, such as a database or tenant. |
| Space-owned | One application environment owns the dependency and its lifecycle. It can serve the same Space across several Cells. Document how placements reach it and how its state is shared, replicated, and recovered. |
| Deployment-local | One logical deployment owns the dependency and coordinates its lifecycle. Include the qualified deployment in the resource identity where needed to distinguish installations. |

Use one logical instance per World and Realm for each dependency that supports suitable Space isolation. Evaluate the dependency’s native boundaries, such as databases, schemas, tenants, namespaces, buckets, accounts, queues, or volumes, including when Spaces need different configuration. Document physical members, replication, endpoints, and reachability for multi-Cell placements.

When a dependency cannot safely serve several Spaces, decide whether its lifecycle follows one Space across placements or one deployment. Before choosing a dedicated instance, an isolation exception, a replacement, or removal, explain the unsupported sharing or isolation requirement and get explicit approval from the responsible project decision-maker.

Runtime role determines ownership. A Temporal server can be Realm-owned infrastructure while a Temporal worker container belongs to the Core service deployment for its Space. An infrastructure component may own reusable tooling for deployment-bound processes while the consuming deployment coordinates their lifecycle.

Use the World–Realm scope in shared infrastructure names such as `uncrn-lab-postgres`. Use the Space scope for resources whose lifecycle follows that Space. Include Space and the consuming Service in deployment-local process names such as `uncrn-lab-sbx-core-worker`, qualifying placement wherever names from multiple Cells share a namespace. Logical databases can remain `uncrn_lab_sbx` and `uncrn_lab_tst` regardless of Area or Cell. Verify isolation independently of resource names. Shared cluster-level roles or administrative credentials remain dependency/Realm-owned where applicable. Application data, authentication state, signing material and files remain Space-owned under their actual contracts. Do not reset shared roles or rotate shared credentials as a side effect of provisioning a Space.

Logical separation must enforce application and API isolation. Sharing a process accepts a common failure and maintenance boundary. Validate that each consumer selects its own database/tenant, endpoint and credentials, and that unauthorized cross-Space requests or tokens cannot access another Space's state. Enforce authorization on each boundary independently of namespaces and ports. Shared mail or object fixtures must be selected and removed by exact ownership rather than clearing a service globally.

Stopping, cleaning, resetting or recovering one Space must preserve shared dependencies and every other Space's resources. Shared-service maintenance must identify its affected Spaces and coordinate them. Keep Realm and Space configuration, state records, locks, and lifecycle operations distinct. Enforce each Space operation’s scope even when it can reach a shared dependency.

Back up the selected logical database, associated files, and required protected configuration as a consistent recovery unit. Restore into the Space’s logical resources. Recovering a shared physical volume is a Realm-wide operation that can affect every Space. Record the Realm roles, compatible versions, and other prerequisites the Space restore depends on, and verify whole-Realm recovery separately.

Verification should demonstrate simultaneous operation, rejection of unauthorized cross-Space access, exact fixture cleanup, preservation of neighboring Spaces during reset/recovery and persistence across authorized shared-service restarts. Projects own decisions about data recreation, reset exceptions, products, and ports.

## Examples from implementation

The examples use fictional World and site names. They describe identities and relationships derived from tracked configuration.

### Local development and Dokploy

World `uncrn` configures Spaces `sbx` and `tst` in Realm `lab` on the host installation `(uncrn, lab, hst)` in Area `edg`. Both deployments run their own application processes and use a Realm-owned Compose project for shared dependencies. PostgreSQL provides separate application databases and owners. The shared Temporal server serves separate namespaces, and each Space runs its own worker. A test reset operates on `tst` resources and preserves `sbx` and shared dependencies.

The production configuration uses environment `(uncrn, prd, hq)` and the Dokploy target `(uncrn, prd, dkp)` in Area `dc`. Application resource names use the production Space. Each project owns the topology and dependency configuration behind its target.

### Shared QA, production Cells, and a Vagrant lab

World `orion` configures QA Spaces `qa01`, `qa02`, and `qa03` in Realm `dev` for one target, `(orion, dev, r1a)`, in Area `dc`. Each Space has its own deployment identity.

In Realm `prd`, Space `hq` has targets `(orion, prd, r1a)` and `(orion, prd, r1b)`, both in Area `dc`. Each installation contains application and database hosts. Inventory declares master and replica responsibilities, while the two Cell slugs stay stable through a role transition. Dependency recovery, writer fencing, and public ingress cutover require their own controls.

In Realm `lab`, the Vagrant inventory defines Cells `v1a` and `v1b` in Area `edg`. Each Cell contains separate application and database VMs. The lab deploy configuration selects Space `dbg`. VM provisioning and application deployment are distinct operations.

## Illustrative examples

### Native, Compose, and VM placements

For a personal laboratory, `(uncrn, lab, hst)` can identify a native host installation even when its dependencies run in Compose. Choose `cmp` when the application services themselves run through Compose. A target `(uncrn, lab, vgt)` can represent a Vagrant installation with several VM nodes. Both installations can occupy Area `edg`.

Space `sbx` can deploy to both `hst` and `vgt` while retaining environment identity `(uncrn, lab, sbx)`. Decide explicitly whether these deployments share or replicate state. Use distinct Spaces when the placements represent distinct application environments.

### Kubernetes across Areas

World `uncrn` deploys the application environment `uncrn-prd-hq` to a Kubernetes cluster with Cell URI `uncrn-prd-k8s`. Two workers have server URIs `prd-east-worker01.uncrn` and `prd-west-worker01.uncrn`. The placement configuration associates Areas `east` and `west` with the same Cell, so both workers belong to one deployment target.

### Availability zones and quorum

Two Cells can occupy different physical failure domains. For example, Cell `east` in `room-a` can serve as master while Cell `west` in `room-b` follows as a replica. Both may host Space `hq`. Changing which Cell serves as master preserves both deployment identities.

One distributed Cell can also span several AZs and host one logical deployment of `hq`. With three voters, one per independent AZ, a majority quorum of two can survive one AZ loss. With five voters in five independent failure domains, a majority quorum of three can survive two domain losses. These examples assume a system designed for majority voting. Evaluate latency, correlated failures, fencing, storage durability, replication mode, and each service’s quorum rules independently.
