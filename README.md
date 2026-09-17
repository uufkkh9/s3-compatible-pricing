# object storage: how S3-compatible buckets work, what they should really cost, and a flat $4.90/TB option worth knowing

Most people who search for object storage aren't looking for a computer science lecture. They're trying to figure out one of three things: whether it's the right kind of storage for what they're doing, how it's different from the block or file storage they already use, or why their cloud storage bill keeps growing faster than their data. This article covers all three, plus the one part most guides skip — where egress and request fees hide, and how a flat-rate provider like Sharktech changes that math.

## What object storage actually is

Object storage stores data as **objects** instead of files on a filesystem or blocks on a disk. Each object is a file plus its metadata, identified by a unique key. Objects live inside **buckets**, which are flat containers — think of a bucket as a top-level folder that can hold an unlimited number of objects, addressed by a key like `bucket-name/images/2026/banner.jpg`.

There's no directory hierarchy to traverse and no disk layout to manage. You put an object in, you get it back by its key, and the storage layer handles replication and redundancy behind the scenes. That's why object storage scales the way it does: adding capacity doesn't involve repartitioning anything, and a bucket with 10,000 objects behaves the same as one with 10 billion.

The model has trade-offs worth understanding upfront:

- **Access happens over HTTP through an API**, not through a filesystem mount. Every read or write is a network request, which adds latency compared to a local disk.
- **You don't modify objects in place.** You overwrite them wholesale. High-churn data with lots of small random writes is a poor fit.
- **Data isn't organized for browsing.** It's organized for retrieval by key, which suits applications, backup tools, and scripts more than humans dragging files around in a file manager.

If what you're storing is files that get written once and read many times — images, videos, database dumps, build artifacts, log archives — those trade-offs mostly don't matter. If you're planning to run a database on top of it, stop reading and look at block storage instead.

## Object vs. block vs. file storage: which one you actually need

The three storage types get explained constantly, but the practical version is short:

| Storage type | How it works | Typical latency | Best for |
| --- | --- | --- | --- |
| Block storage | Disk volumes attached to a server | Sub-millisecond | Databases, OS disks, VMs |
| File storage | Folders and files over a network (NAS) | Low, but depends on protocol | Shared drives, home directories |
| Object storage | Flat buckets of objects accessed over an API | Roughly 5–50 ms per request | Backups, media, archives, app assets |

Block storage is what you want when an application needs to do lots of small, fast random reads and writes — a Postgres database, an email server, a root filesystem. Object storage adds an HTTP round trip to every operation, so it can't compete on raw latency, and it isn't supposed to.

File storage sits in between: it preserves the folder structure people are used to, which makes it convenient for humans and awkward for scaling past a few hundred terabytes.

Object storage wins on three axes the others don't touch: virtually unlimited scale, no capacity pre-planning, and — usually — a much lower price per terabyte. That's why it has quietly become the default answer for anything that isn't a running database or an interactive filesystem.

## S3 compatibility: the feature that matters most

Here's the part beginners often miss. "Object storage" as a product category is dominated by one specific API: **Amazon S3**. S3 became the de facto standard, and nearly every competing object storage service — including Sharktech's — implements the S3 API so existing tools work with nothing more than an endpoint and credential swap.

What that means in practice: if a tool can talk to AWS S3, it can talk to any S3-compatible provider. That includes:

- Backup tools like **restic** and **rclone**, both of which support S3 backends natively
- CI/CD tooling — Jenkins, GitLab, Terraform pipelines that push build artifacts to storage
- The official AWS SDKs and CLI, `aws-cli` included, which work against any S3-compatible endpoint once you point them elsewhere
- Content platforms and web apps that serve media directly out of a bucket

This is why S3 compatibility is arguably a bigger purchasing criterion than any individual feature. The provider you pick matters less than whether your existing toolchain connects without custom integration work.

## The pricing trap: where your bill actually grows

The advertised storage rate is rarely what sinks your budget. The classic pattern looks like this: a provider charges a reasonable per-GB rate for storage, then bills separately for **egress** (data leaving their network), API requests, retrieval, and early-deletion penalties. A cheap-looking bucket quietly turns into a three-figure invoice.

The math is not subtle. At AWS S3's published rates, standard storage runs about $0.023 per GB — roughly $23/TB/month — while egress costs around $0.09 per GB. Pull one terabyte of your own data back out and that's another ~$90 on top of the storage charge. Restore 10TB after an incident and you've spent $900 for the privilege of accessing files you already paid to store.

This is why the honest way to compare providers is storage + egress + request fees + minimum commitments, added together. A few patterns dominate the current market:

- **AWS S3**: enormous ecosystem, premium pricing, meaningful egress charges
- **Backblaze B2**: about $6.95/TB/month, with free egress up to 3× your stored data
- **Cloudflare R2**: around $15/TB/month, with egress at zero
- **Wasabi**: competitive storage rates, but with a 90-day minimum storage commitment — you keep paying for data even if you delete it early

Each model fits a different data shape. The one thing all of them share is that the headline number isn't the whole number.

## Sharktech's approach: one flat rate, and nothing else on the invoice

Sharktech has been in the infrastructure business for over two decades, with data centers in Los Angeles, Denver, Chicago, and Amsterdam. Their S3 Object Storage service runs on redundant clusters inside those facilities and is available across their network, with 40G inbound and outbound connectivity — meaning the practical bottleneck for upload and download speeds is almost always the client's side, not the cluster.

The pricing philosophy is unusually blunt: **storage and bandwidth are the only items that ever show up on the invoice.** No per-request fees, no storage-class transitions, no contract lock-in. The published rate is $4.90 per TB per month, with bandwidth listed at $0.00 on the pricing card, and the same flat rate applies whether you store a little or a lot.

Here's the current official pricing structure:

| Plan | Storage | Bandwidth (egress) | Billing | Contract | Order |
| --- | --- | --- | --- | --- | --- |
| S3 Object Storage — 1TB | 1TB | 1TB listed at $0.00 | $4.90/month | None, cancel anytime | [Get 1TB of S3 storage for $4.90/month](https://bit.ly/SharKTech) |
| S3 Object Storage — 5TB | 5TB (at the flat $4.90/TB rate) | Included, billed only as a separate bandwidth line if exceeded | $24.50/month | None | [Order 5TB at the flat rate](https://bit.ly/SharKTech) |
| S3 Object Storage — 10TB | 10TB (at the flat $4.90/TB rate) | Included, billed only as a separate bandwidth line if exceeded | $49.00/month | None | [Order 10TB at the flat rate](https://bit.ly/SharKTech) |
| Custom S3 plan | Custom storage volume | Custom | Contact for quote | None | [Request a custom storage plan](https://bit.ly/SharKTech) |

The 5TB and 10TB figures are the flat rate applied consistently — Sharktech's published position is that the per-TB price doesn't change with volume, so 10TB works out to $49/month. Compare that to roughly $230/month for the same 10TB on hyperscaler standard storage, before egress, and the gap becomes the entire reason to look at this category.

> **The key details:** S3 API compatible, no long-term commitment, redundant clusters in Sharktech's own data centers, and direct access to support via email and phone rather than a ticket maze.

If that flat-rate model fits your workload, you can 👉 [check Sharktech's S3 Object Storage pricing and order directly](https://bit.ly/SharKTech) — the service is available for immediate deployment.

## How it stacks up against the usual recommendations

Backblaze B2 is the default budget recommendation in this space, and legitimately so: $6.95/TB with free egress up to 3× stored data is a solid deal. Sharktech undercuts the storage rate at $4.90/TB. If your workload transfers heavily — CDN origins, frequent restores, download-heavy apps — the difference compounds every month.

Cloudflare R2's zero-egress pitch is unbeatable when outbound transfer dominates your costs, but its storage rate is roughly triple Sharktech's. A useful rule of thumb: **store a lot, transfer a little — the flat low rate wins. Store a little, transfer constantly — R2 wins.**

Against Wasabi, the deciding factor is the 90-day minimum commitment. For short-lived data — staging artifacts, rotating uploads, temporary backups — that commitment makes Wasabi's effective rate higher than advertised. Sharktech has no such minimum, which matters if your data doesn't stick around.

And against AWS S3 itself, this isn't a fair fight on price. AWS wins on ecosystem breadth, edge locations, and storage-class variety. If you need those things, you're paying for them on purpose. If your goal is cheap, predictable, S3-compatible storage, you're buying capabilities you may never use.

One honest caveat: Sharktech's object storage offering is newer than its two-decade hosting track record, and it doesn't have the massive third-party benchmarking history the hyperscalers do. The flat-rate pricing and S3 compatibility are the draw; the service is best evaluated against your own workload rather than someone else's benchmark.

## What object storage is good for — and what it isn't

Based on the use cases where S3-style storage genuinely performs:

1. **Backups and archives.** This is the sweet spot for flat-rate pricing. Pipe nightly backups from restic, rclone, or similar tools into a bucket, and two things matter: cost per TB, and what a full restore costs you. With bandwidth listed at $0.00, disaster recovery doesn't turn into a second disaster.
2. **Media and user-generated content for web apps.** S3 integrates cleanly with HTML and JavaScript, so serving images, videos, and uploads straight from a bucket is a standard pattern. A traffic spike in downloads shouldn't generate a surprise transfer bill.
3. **DevOps artifacts and CI/CD storage.** Build artifacts, deployment packages, release tarballs — S3 slots into Jenkins, GitLab, and Terraform workflows natively, and usage-based flat pricing means a busy release week doesn't require capacity planning.
4. **Long-term compliance and legal retention.** Infrequently accessed data that must be retained benefits from redundancy and low cost without hyperscaler premiums for reads you'll rarely perform.

What it's *not* for: databases, anything doing high-churn small writes, workloads that need sub-millisecond latency, or teams that genuinely need folder-hierarchy browsing for day-to-day human work. If your use case reads like that list, block or file storage will serve you better regardless of price.

## Getting started

The practical setup path is the same for any S3-compatible provider:

1. Create an account on the provider's portal and order the storage amount you need — for Sharktech, that's the 1TB configuration at $4.90/month, scalable at the same flat rate.
2. Create a bucket in the portal (or via the API).
3. Generate an access key and secret for the S3 endpoint.
4. Point your existing tooling at the endpoint — in rclone, that's an S3 backend config; in restic, an `s3:` repository URL; in the AWS CLI, a custom endpoint setting.

Since the service speaks the standard S3 API, there's no proprietary client to learn. Anything that already works against S3 works here after swapping the endpoint and credentials.

Ready to test it against your own data? 👉 [Order Sharktech S3 Object Storage at $4.90/TB and start with 1TB](https://bit.ly/SharKTech) — and if your storage needs are larger or unusually shaped, their support team handles custom plans directly.
