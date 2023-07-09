---
title: Creating a BlackHole Shell with 16 Lines of Rust Code
layout: post
author: Zola Gonano
---
I have been using [rbash](https://en.wikipedia.org/wiki/Restricted_shell) to restrict users' shell access for the past year, and there have been some problems with it that finally led me to write my own shell. The biggest problem was that users could run [bash](https://www.gnu.org/software/bash/), [dash](http://gondor.apana.org.au/~herbert/dash/), or [zsh](https://www.zsh.org/), or any other shell, allowing them to bypass the restrictions. Another issue was the ability for users to execute unauthorized software, which was not restricted enough for my requirements. Therefore, I decided to develop a black hole shell.

The way it works is quite simple: it takes everything and does nothing with it.

To make it, I will be using the [Rust programming language](https://www.rust-lang.org/), which is known for its memory safety and relative ease of use compared to other languages offering similar performance. Additionally, I will utilize the [ctrlc](https://crates.io/crates/ctrlc/) crate to handle Ctrl+C signals.

# Setting Up the Crate

First, we need to create a binary crate using the `cargo new` command and add our dependencies. Then we can proceed with writing the shell.

Create the binary crate:

```bash
cargo new bhsh --bin
```

After running the above command, Cargo will generate a directory named `bhsh` with the necessary files and structure for our project. Next, navigate to the crate directory:

```bash
cd bhsh
```

Add `ctrlc` as a dependency:

```bash
cargo add ctrlc
```

Once we have finished setting up our crate and adding the necessary dependencies, we can proceed to implement the actual shell.

# Implementing the shell

The implementation part is pretty straightforward. We need to create a loop that takes user input and ignores it.

First thing we will import the modules needed to top of our `src/main.rs` file:

```rust
use ctrlc::set_handler;
use std::io::{self, BufRead};
```

Then we write the `main` function:

```rust
use ctrlc::set_handler;
use std::io::{self, BufRead};

fn main() {
    let _ = set_handler(move || { });

    let stdin = io::stdin();
    let mut stdin_lock = stdin.lock();
    let mut input = String::new();

    loop {
        let _ = stdin_lock.read_line(&mut input);
        input.clear();
    }
}
```

The first line inside the function simply sets a handler for Ctrl+C signals that does nothing. Therefore, when the user hits Ctrl+C, it won't break the program. We then create a listener for stdin to take inputs from the user. Inside the loop, we retrieve the user's input and do nothing with it.

This shell is available on my [GitHub](https://github.com/zolagonano/bhsh) as well as [crates.io](https://crates.io/crates/bhsh), and all contributions to it are welcome.

---

I should mention that I'm not a security expert, and I'm unsure whether this shell is inherently safer than others. I would appreciate any feedback or insights if you believe there are technical, grammatical, or any other issues in this blog post or my other posts. Feel free to open an issue on [GitHub](https://github.com/zolagonano/awesome-zeronet) or email me.
