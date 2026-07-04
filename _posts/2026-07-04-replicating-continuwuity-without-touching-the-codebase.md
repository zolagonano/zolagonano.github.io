---
title: "Replicating Continuwuity without touching the codebase"
author: Zola Gonano
layout: post
---

Personally I am not a fan of RocksDB because of it being such a headache to get basic things working like replication, clustering, high availability. But I recently used continuwuity to host a matrix homeserver hosting thousands of users (it never meant to grow that much) and the whole time I had this fear of losing everyone's data because I had no replication or backups of the database and no reliable way of doing so.

What couldn't have been done?

**Omnipaxos**  
Omnipaxos is an interesting tool. It is a consensus layer that sits in between your program and your rocksdb database allowing you to have real time replications and high availability clustering.

But this required changing continuwuity's codebase which meant hours of reading code and changing and taking a huge risk swapping the homeserver to an untested modified version in production like this.

So this option wasn't really a good one for this case.

**Rocksplicator**

Rocksplicator is a tool developed by Pinterest to provide replication and clustering for RocksDB database but the documents weren't very clear and I simply couldn't get it running.
But logically this might be one of the better options for this job if you get it running properly.

**Lsyncd**  
Lsyncd is a tool that watches for file changes and transfers the changes over to the specified destination using rsync.

This is perfect for a lot of syncing tasks but as long as the files aren't constantly opened and changed since that would cause corrupted data on the destination and repairing rocksdb database isn't much of an easy task either.

Although I ended up using lsyncd for media storage of the homeserver and the database without causing corruption.

**What worked?**  
RocksDB database doesn't allow multiple mutable connections but it can be opened as read-only alongside another connection, and it also has a feature named checkpointing.

Checkpointers are essentially a checkpoint in time in which the database can be restored to without any corruption. And it does that using links in the filesystem so it doesn't generate new copies of the database every time a checkpoint is created.
This keeps every original wording and sentence structure intact while fixing spelling, grammar, capitalization consistency, and minor punctuation issues.
And Lsyncd can be used here to detect the corresponding files to the links and transfer the checkpoints to the destination.

But there was a catch. This couldn't be done in real time and had to be done periodically. Which is fine in this case, being a few minutes behind in a matrix homeserver isn't such a big issue and the server will eventually catch up after some time.

**The execution**  
To execute the idea I decided to go with rust and wrote a simple program that creates a checkpoint and preserves the previous checkpoint (in case of incidents during checkpointing) every time it's executed.

So it's a very basic tool, it's nothing fancy.

```rust
use rocksdb::{ColumnFamilyDescriptor, DB, Options, checkpoint::Checkpoint};

use std::env;
use std::fs;
use std::path::PathBuf;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        eprintln!("Usage: {} <db_path> <checkpoint_root>", args[0]);
        std::process::exit(1);
    }

    let db_path = &args[1];
    let backup_root = &args[2];

    let mut opts = Options::default();
    opts.create_if_missing(false);

    let cf_names = DB::list_cf(&opts, db_path)?;

    let cf_descriptors: Vec<_> = cf_names
        .iter()
        .map(|name| ColumnFamilyDescriptor::new(name.clone(), Options::default()))
        .collect();

    fs::create_dir_all("/tmp/rocksdb_secondary")?;
    fs::create_dir_all(backup_root)?;

    let db = DB::open_cf_descriptors_as_secondary(
        &opts,
        db_path,
        &String::from("/tmp/rocksdb_secondary"),
        cf_descriptors,
    )?;

    db.try_catch_up_with_primary()?;

    let root = PathBuf::from(backup_root);

    let current_dir = root.join("checkpoint_current");
    let prev_dir = root.join("checkpoint_prev");
    let staging_dir = root.join("checkpoint_staging");

    if staging_dir.exists() {
        fs::remove_dir_all(&staging_dir)?;
    }

    let checkpoint = Checkpoint::new(&db)?;
    checkpoint.create_checkpoint(&staging_dir)?;

    if prev_dir.exists() {
        fs::remove_dir_all(&prev_dir)?;
    }

    // this is to prevent lsyncd from re uploading everything
    if current_dir.exists() {
        fs::rename(&current_dir, &prev_dir)?;
    }

    fs::rename(&staging_dir, &current_dir)?;

    println!(
        "Checkpoint created successfully at: {}",
        current_dir.display()
    );

    Ok(())
}
```

Then the current_checkpoint directory is synced to the destination server using lsyncd. When syncing with rsync it needs them to convert the hardlinks into actual files. In rsync it can be done by using the -a and --no-hard-links args.

Also I wrote a systemd service and timer to run the checkpointer periodically:

**/etc/systemd/system/checkpointer.service**

```toml
[Unit]
Description=Conduwuit RocksDB Checkpointer

[Service]
Type=oneshot
ExecStart=/usr/local/bin/checkpointer /var/lib/conduwuit/db /var/lib/conduwuit/checkpoints
LimitNOFILE=1048576 # Increase if you have a very large database

# retry checkpointing if a file is still used by primary connection
Restart=on-failure
RestartSec=30

StartLimitBurst=10
```

**/etc/systemd/system/checkpointer.timer**

```toml
[Unit]
Description=Run checkpointer on timer

[Timer]
OnBootSec=10min
OnUnitActiveSec=30min # Increase
Persistent=true

[Install]
WantedBy=timers.target
```

It could've been done with tokio and timers in the rust code itself but I didn't want to bloat the code with a functionality that already exists in linux (Also cronjobs would've worked fine too).

---

The codes are also available on my GitHub at [github.com/zolagonano/rocksdb_checkpointer](https://github.com/zolagonano/rocksdb_checkpointer) and all contributions on it are welcome and appreciated.
