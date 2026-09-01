---
name: cicd-pipeline
description: Method and checklists for shipping a containerized application with GitHub Actions, Docker and Kubernetes — build once, publish a signed immutable image, promote that exact digest through gated environments to production. Covers the inventory to gather first, the dev/infra boundary, build-once/promote-by-digest, supply-chain integrity, deployment strategies, secrets, health-gated rollout, GitHub Environments approvals, and rollback. Use when asked to create or configure a pipeline, onboard an app to CI/CD, set up an environment, promote to production, or migrate a hand-run deploy to an automated one.
---

# CI/CD pipeline — GitHub Actions, Docker, Kubernetes

Build one immutable, signed image in CI; promote that exact image by digest through environments in CD, each stage gated by checks and — for production — a GitHub Environment approval. Details differ per app; the sequence, the boundaries and the failure modes do not.

- **CI/CD** — GitHub Actions. **Registry** — GHCR (`ghcr.io/<org>/<app>`). **Build** — `docker buildx`.
- **Deploy target** — a Kubernetes cluster (Deployment + Service + Ingress, via Helm or Kustomize). A single-host Docker Compose target is the lightweight option.

## Reference architecture

- **Delivery config separate from app code** — a `deploy/` folder or a dedicated repo. Reason: different change cadence and review rules. It holds the workflows and per-environment values.
- **Build once, promote by digest.** One image per release candidate, identified by content digest (`@sha256:…`). The same digest flows staging → production — never rebuilt or re-tagged per environment. Tag with the version (`X.Y.Z`), pin by digest in every manifest. Environment is a deploy-time input.
- **Promotion, not branches.** Trunk-based: one `main`, every merge is a release candidate, promoted through workflow stages gated by tests, policy and approvals. Environment chosen by a workflow input — one parameterized workflow, not drifting copies. *(Branch-per-environment works too; map the stages onto branches.)*
- **Keyless auth.** Workflows reach the cluster and cloud APIs via GitHub OIDC, not stored long-lived credentials. GHCR uses `GITHUB_TOKEN` with `packages: write`.
- **Scoped identity.** Each deploy identity is bound to one environment and expires. No human has standing write access to production; changes go through the pipeline. Break-glass is pre-approved, logged, time-boxed.
- **Version marker:** a signed git tag `X.Y.Z` (SemVer). CI and CD fail closed when it is absent.

## Core rules

1. **Inventory before any file.** No manifest, workflow or secret declaration without the full dependency inventory (next section). Missing item → list it and **stop**. Never infer a value from a template default or another app.
2. **Zero assumptions about real config.** Ports, DB hosts, domains, credentials, sizing, versions — confirmed with whoever operates the target. When migrating, every inherited value is a candidate to confirm, not copy.
3. **Dev/infra boundary.** The `Dockerfile` and source belong to the developer — do not edit them. Manifests, workflows, secrets, networking, ingress, DNS belong to infra. A build/code failure → report and wait; do not work around it in the pipeline. For a framework-level issue, say **which value is missing** and **which layer owns it** (manifest = infra; `Dockerfile`/app config = dev), then hand the check back.
4. **Build once, promote by digest.** The image validated in staging is bit-for-bit the one that reaches production.
5. **Supply-chain integrity.** CI generates an SBOM, signs the image, attaches provenance (GitHub artifact attestations); vulnerability and secret scanning are **blocking** gates; the cluster verifies signature and provenance before admitting the image (admission policy).
6. **No secret in plaintext** — not in a repo, manifest, workflow, build arg, or image layer. Prefer runtime injection into the workload over writing secrets to disk or the job environment.
7. **Immutable infrastructure.** Config change → redeploy through the pipeline, never a live `kubectl edit`/`exec` in production. Roll the whole workload.
8. **Validate before commit.** Render every manifest with values resolved and lint it (`helm template`/`lint`, `kustomize build`, `kubectl apply --dry-run=server`, `docker compose config`); fail on any unresolved required value.
9. **Health-gated, progressive delivery.** CD reports success only when the new version serves: readiness passed, `kubectl rollout status` converged, smoke test green, error/latency SLOs held. Any failure → automatic rollback. In production use a strategy that limits blast radius (Part 1).
10. **Production is gated and audited.** A GitHub Environment with required reviewers and a freeze policy stands between a green build and production; the approval and outcome are recorded.
11. **Schema changes are backward-compatible (expand/contract).** Old and new code both run against the DB during a rollout; migrations are decoupled from the code deploy and safe while the previous version still serves; never destructive in the same release that stops using the column.
12. **Rollback is a first-class path**, exercised, run **through the pipeline** (redeploy the previous digest) — not a manual cluster edit.
13. **Observe the deploy.** Emit a deployment marker; track DORA metrics; correlate alerts with deploys.
14. **Document on completion.** Record the app in the wiki (delivery repo, environments, identities, strategy, rollback command) and every deviation.

## Dependency inventory

Per application, and per environment where values differ:

- Cluster + namespace; deploy path (OIDC role, `kubeconfig` context)
- Every workload (Deployment / StatefulSet / CronJob) and per workload: image (own vs upstream), ports, env vars, mounted config, command/args
- Secrets: which values, where consumed, which store holds them
- External dependencies with **real** addresses (DB, cache, broker, SSO/LDAP, object storage, APIs) — host, port, DB/bucket
- Persistence: PVCs, storage class, size, backup/restore expectation
- Networking: who talks to whom, internal vs exposed, NetworkPolicy
- Exposure: domain, public URL, Ingress class, TLS termination and cert source, DNS record
- Health: readiness/liveness/startup endpoints, expected response, startup time, SLOs to gate on
- Resources: requests, memory limit, replica count, HPA bounds, PodDisruptionBudget
- Deployment strategy and its parameters; previous known-good digest and where it is recorded

Partial inventory → stop.

## Placeholders

`<org>`/`<repo>` · `<env>`/`<ENV>` · `<image>` = `ghcr.io/<org>/<app>` · `<digest>` = `sha256:…` from CI · `<workload>` = Deployment name · `<namespace>` · `<SECRET>` = name identical to its manifest reference.

---

## Common pitfalls

**Manifest**
- **Indentation** — a misindented `env:`/`volumeMounts:`/`resources:` attaches to the wrong object or is dropped silently (compose: `networks:`/`volumes:` under `services:` become a bogus service). Render and diff.
- **Unresolved reference** — a value with no binding resolves empty; the workload starts without the credential. Fail the render.
- **Mutable image reference** — `latest`, a floating minor, an overwritable tag. Pin by digest.
- **Isolation** — wrong namespace, or missing `Service`/`NetworkPolicy` → unreachable.
- **Naming drift** — inconsistent names/labels, or a leftover `-staging` string in production values.
- **No `securityContext`** — baseline: `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, drop `ALL` caps, `seccompProfile: RuntimeDefault`.
- **Probe mistakes** — no readiness; liveness and readiness on the same dependency-heavy endpoint (a downstream outage restart-loops every pod); no startup probe for a slow starter.
- **Single replica / no PodDisruptionBudget** — a node drain takes the service down.

**Workflow**
- Third-party actions pinned to a tag or `@main`, not a commit SHA.
- Over-broad `permissions:`; secrets exposed to untrusted PR code (`pull_request_target` + PR-head checkout).
- Misaligned step blocks (`env:`, `with:`).
- Build+publish for an upstream image (unneeded), or missing for one you own.
- Environment selected by drifting copy-pasted workflows instead of one parameterized workflow with an `environment:` input.
- OIDC role / cluster context not scoped to the target environment.
- CD on `on: push` instead of after the previous stage's gate.

**Deploy execution**
- Success reported on `kubectl apply` without `kubectl rollout status`.
- A non-backward-compatible migration in the same release as the code; a migration Job run non-blocking.
- CD re-deriving the version from git instead of consuming the digest CI produced.
- Auth not refreshed each run — an expired OIDC token fails with `unauthorized`.

---

## Part 1 — Building the pipeline

Once per application: produces the parameterized workflow plus one environment (typically staging). Adding production is Part 2.

### 1.1 Prerequisites
- App repo; `main` protected (PR + review, required status checks, linear history)
- `deploy/` folder (or delivery repo), same protection
- GHCR enabled; `GITHUB_TOKEN` `packages: write` in the build job
- Per environment: a namespace + scoped ServiceAccount/RBAC, reached from Actions via an OIDC role
- A secret store for real secrets only; non-sensitive values stay in the manifest
- Cluster admission policy: verify signature + provenance, enforce Pod Security "restricted", forbid mutable tags
- GitHub Environments (`staging`, `production`); `production` with required reviewers and a freeze policy
- Observability: a deployment-marker integration; a dashboard and SLO alerts

### 1.2 Layout

```
.github/workflows/ci.yml    build, scan, sign, publish — emits digest + version
.github/workflows/cd.yml    reusable; inputs: environment, digest
deploy/
  base/                     Helm chart or Kustomize base
  envs/<env>/values.yaml    per-environment overrides only
```

### 1.3 Deployment manifest

From the developer's reference manifest, produce `base/` + `envs/<env>/`:

- Image `<image>@${digest}` — digest supplied at deploy time, never an inline build or mutable tag
- `securityContext` restricted; readiness + liveness + startup probes distinct; requests set, memory limit = request, no CPU limit
- Replica count, HPA and PodDisruptionBudget on every stateless workload
- Ports per the inventory; secrets via `secretKeyRef` from a `Secret` synced by the store, never inlined
- `Service`/`Ingress`/`PVC` at the right scope; `NetworkPolicy` for anything not fully public
- Remove dev-only config (debug, hot-reload, local hosts)
- The per-environment file holds only what differs (replicas, resources, hostnames)
- Validate: `helm template` / `kustomize build` + `kubectl apply --dry-run=server`

### 1.4 CI (`ci.yml`)

Steps, in order: trigger on merge to `main` and tag `v*` → checkout at the tag → resolve the version from the signed tag, exit non-zero if none → `docker login ghcr.io` with `GITHUB_TOKEN` → `docker buildx build --push` to `<image>:X.Y.Z`, capture the digest from `--metadata-file` → generate SBOM → scan image + deps + secrets, **block** on policy violations → sign the digest and attach provenance → set `digest` and `version` as job outputs.

```yaml
permissions: { contents: read, packages: write, id-token: write, attestations: write }
# key steps
- id: ver
  run: |
    TAG=$(git describe --tags --exact-match) || { echo "no release tag"; exit 1; }
    echo "version=$TAG" >> "$GITHUB_OUTPUT"
- id: build
  uses: docker/build-push-action@<sha>
  with: { push: true, tags: "ghcr.io/${{ github.repository }}:${{ steps.ver.outputs.version }}" }
- uses: actions/attest-build-provenance@<sha>
  with: { subject-name: "ghcr.io/${{ github.repository }}", subject-digest: "${{ steps.build.outputs.digest }}" }
```

### 1.5 CD (`cd.yml`, reusable)

Inputs: `environment`, `digest` (from CI — never re-derived). Steps: `environment:` on the job so GitHub enforces reviewers/rules → assume the environment's OIDC role, set the `kubeconfig` context → backward-compatible migration Job if any (apply, wait, abort on failure) → deploy with the digest pinned → wait for convergence + health, roll back on failure → smoke test by the service address → emit the deployment marker.

```yaml
on: { workflow_call: { inputs: {
  environment: { type: string, required: true }, digest: { type: string, required: true } } } }
permissions: { id-token: write, contents: read }
jobs:
  deploy:
    environment: ${{ inputs.environment }}          # GitHub enforces reviewers / rules
    steps:
      - uses: actions/checkout@<sha>
      - uses: <helm-setup-action>@<sha>             # + your cloud's OIDC login action for the kubeconfig
      - run: |
          helm upgrade --install <workload> deploy/base -n <namespace>-${{ inputs.environment }} \
            -f deploy/envs/${{ inputs.environment }}/values.yaml \
            --set image.digest=${{ inputs.digest }} --wait --timeout 5m --atomic
          kubectl rollout status deploy/<workload> -n <namespace>-${{ inputs.environment }} --timeout=5m
```

### 1.6 Wire the stages
`ci.yml` on merge to `main` → on success call `cd.yml` with `environment: staging` (automatic) → on staging success call `cd.yml` with `environment: production` (the Environment's required reviewers gate it).

### 1.7 Deployment strategy (production)

Pick by: *can two versions run at once?* and *is there a metric to gate on?*

- **Rolling** (default, stateless) — `RollingUpdate`, `maxSurge: 25%`, `maxUnavailable: 0`. Needs backward-compatible changes.
- **Recreate** — `strategy: Recreate`. Only when two versions cannot coexist; accepts downtime.
- **Blue-green** — a second Deployment behind the same `Service`; flip the selector once healthy; keep the old for instant rollback. ~2× resources briefly.
- **Canary** — a small second Deployment + a weighted `Ingress` split; grow the weight while watching error rate / latency, abort on regression. Strongest gate; needs reliable metrics and, in practice, a progressive-delivery controller for the analysis.

### 1.8 External exposure
- DNS → the Ingress controller's load balancer
- `Ingress` in `deploy/` with the environment's class and annotations
- TLS from an automated in-cluster certificate issuer, not a hand-placed key
- Expose only the intended host/path; everything else `ClusterIP` + `NetworkPolicy`

### 1.9 Cutover (only when migrating an existing deployment)
Deploy the new pipeline's build to a new namespace alongside the old stack → validate → shift traffic (DNS / Ingress weight) gradually, keeping the old stack until stable → decommission the old stack and its manual access.

### 1.10 Post-deploy verification
- Manifest renders clean; running pods report the expected digest
- Each workload answers on its port; readiness/liveness/startup healthy; `rollout status` clean
- Auth; DB and external-service access; data-writing paths; CronJobs
- Secrets resolved at runtime (nothing started empty); no plaintext secret on a pod filesystem
- Signature enforcement works — an unsigned image is rejected
- Logs and metrics reach the sinks; the deployment marker landed
- PDB present; the rollout survives a node drain; previous known-good digest recorded

**Reviewing an existing set:** diff against another environment's config and walk 1.3–1.6 + the pitfalls; the checks not visible in files are the protected branch, the gated Environment, per-environment secrets and OIDC trust, and the cluster admission policy.

---

## Part 2 — Adding production

Production is another call of `cd.yml`, not a copy.

**Reused:** the workflows, the GHCR package and OIDC trust, the image (same digest), the smoke-test suite.

**Add:**
- A production namespace + a separate scoped OIDC role, never staging's
- Production secrets — **production values differ from staging's; never reuse one**
- The `production` GitHub Environment: required reviewers, deployment branch/tag rule, freeze calendar
- `deploy/envs/production/values.yaml`: replica count and sizing for real load (≠ staging), production hostnames, HPA bounds, PDB
- The production deployment strategy (1.7) and its analysis metrics
- Alert routing and on-call

**Confirm real production values** (do not copy from staging): DB server/instance/port/name; integration hosts; domain and public URL; certificate source; replicas, sizing, HPA bounds; canary metrics. Each confirmed with whoever operates production.

**Validate:** run the review; verify the Environment approval actually blocks; verify signature enforcement and Pod Security in the production cluster; rehearse a rollback in staging first.

---

## Part 3 — Secrets

**Order of preference:** (1) a `Secret` synced from your cluster/cloud secret store by an operator or CSI driver — the value never touches Actions, rotation is automatic; (2) the workflow writes a `Secret` at deploy time (GitOps: commit a sealed/encrypted reference only); (3) the workflow injects the value into the deploy step's environment. Never (4) plaintext in a repo/manifest/image.

- **Runtime injection (preferred):** a secrets-store CSI driver or a secret-sync operator pulls from the store and materializes a `Secret`; the manifest references it by `secretKeyRef`/`envFrom`. Nothing sensitive in CI.
- **Workflow-mediated (fallback):** authenticate to the store via OIDC in the deploy job, then `kubectl create secret generic <name> --from-literal=<SECRET>="$VALUE" --dry-run=client -o yaml | kubectl apply -f -`, or inject at the step (`env:` with `${{ secrets.<NAME> }}`) only for values the render needs as env vars.
- **Rules:** name identical to the manifest reference; least-privilege read; never `echo` it or put it in a step name; never a Docker `build-arg` (it lands in a layer); rotate on schedule and on suspected exposure; keep `${{ secrets.* }}` out of untrusted-PR workflows.
- **TLS / file secrets:** prefer an automated in-cluster issuer so no static key exists. Otherwise pull it at deploy time into a `Secret` mounted `readOnly`, `defaultMode: 0400`; never commit or bake it in; rotate.

---

## Part 4 — Operation

**Triggering:** CI on merge to `main` (release candidate) or a signed tag `X.Y.Z` (release), fail closed with no tag. CD staging automatic on green CI. CD production behind the Environment's required reviewers + deployment rule + freeze check.

**Rollback** — through the pipeline, the same mechanism as a deploy, so the audit trail holds and the next deploy does not re-ship the bad version:

- **Helm** — `helm rollback <release> <revision>` as an immediate stop-gap, then re-run `cd.yml` pinning the previous digest.
- **Kustomize / raw** — `kubectl rollout undo deploy/<workload>` stop-gap, then re-run `cd.yml` with the previous digest.
- **`--atomic`** — a failed `helm upgrade` already rolled itself back; the job just fails loudly.
- **Blue-green / canary** — flip the selector back / drop the canary weight to zero; the stable set never left.
- **Compose** — `DIGEST=<previous> docker compose … up -d --wait`.

The previous known-good digest comes from the last successful production deploy (every published digest is in GHCR). Verify convergence and health after a rollback as for a deploy, and open a fix-forward — a rollback is a pause, not a resolution.
